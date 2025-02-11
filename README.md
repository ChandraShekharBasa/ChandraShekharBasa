***************************
APPLICATION FAILED TO START
***************************
 
Description:
 
Field commonUtilityWebClient in com.jpmc.cb.cbuo.commercial.card.service.impl.CommonUtilityServiceImpl required a bean of type 'org.springframework.web.reactive.function.client.WebClient' that could not be found.
 
The injection point has the following annotations:
	- @org.springframework.beans.factory.annotation.Autowired(required=true)
 
 
Action:
 
Consider defining a bean of type 'org.springframework.web.reactive.function.client.WebClient' in your configuration.


package com.jpmc.cb.cbuo.commercial.card.service.impl;

import com.google.gson.Gson;
import com.jpmc.cb.cbuo.commercial.card.common.constant.CommonConstants;
import com.jpmc.cb.cbuo.commercial.card.exception.DownstreamServicesException;
import com.jpmc.cb.cbuo.commercial.card.model.clientdata.ClientSearchResponse;
import com.jpmc.cb.cbuo.commercial.card.model.clientdata.ClientSystemAdmin;
import com.jpmc.cb.cbuo.commercial.card.model.clientdata.InviteDetailsResponse;
import com.jpmc.cb.cbuo.commercial.card.model.employeesearch.EmployeeDTO;
import com.jpmc.cb.cbuo.commercial.card.model.employeesearch.EmployeeSearchResponse;
import com.jpmc.cb.cbuo.commercial.card.model.partycentral.AlternateIdentifiers;
import com.jpmc.cb.cbuo.commercial.card.model.partycentral.Party;
import com.jpmc.cb.cbuo.commercial.card.model.partycentral.PartySearchClientResponse;
import com.jpmc.cb.cbuo.commercial.card.model.partycentral.external.ReferenceData;
import com.jpmc.cb.cbuo.commercial.card.model.wfds.FacilityInfo;
import com.jpmc.cb.cbuo.commercial.card.model.wfds.OutstandingType;
import com.jpmc.cb.cbuo.commercial.card.model.wfds.WFDSResponse;
import com.jpmc.cb.cbuo.commercial.card.model.wfds.WFDSSnapshotResponse;
import com.jpmc.cb.cbuo.commercial.card.service.CommonUtilityService;
import com.jpmc.cb.cbuo.commercial.card.service.EmployeeMapper;
import lombok.extern.slf4j.Slf4j;
import org.apache.commons.lang3.StringUtils;
import org.slf4j.MDC;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.http.*;
import org.springframework.stereotype.Service;
import org.springframework.util.CollectionUtils;
import org.springframework.web.client.RestTemplate;
import org.springframework.web.reactive.function.client.WebClient;
import reactor.core.publisher.Mono;

import java.time.Duration;
import java.util.ArrayList;
import java.util.List;
import java.util.*;
import java.util.concurrent.atomic.AtomicReference;

@Service
@Slf4j
public class CommonUtilityServiceImpl implements CommonUtilityService {

    @Autowired
    private RestTemplate cbuoRestTemplate;

    @Value("${cbuo-common-utility-svc.baseUrl:https://cbuo-common-utility-svc.dev.aws.jpmchase.net/v1/}")
    private String cbuoUrl;

    @Autowired
    @Qualifier("commonUtilityWebClient")
    private WebClient commonUtilityWebClient;

    @Value("${common-utility-timeout}")
    private int timeoutDurationInMilliseconds;

    @Override
    public List<ClientSystemAdmin> getSystemAdminDetails(String ecid) {
        String url = cbuoUrl + ecid + "/invite-details";
        return getCBUOResponse(ecid, url);
    }

    @Override
    public ArrayList getPartyCentralDetails(String ecid) {
        String url = cbuoUrl + "api/party/clientSearch?ecid=" + ecid;
        return getCBUOPartyClientResponse(ecid, url);
    }

    public String geFacilityStatus(String ucn) {
        String creditFacilityStatus = "";
        WFDSResponse wfdsResponse = getClientFacilityList(ucn);
        List<FacilityInfo> facilityInfoList = getFacilityInfo(wfdsResponse);
        if (!CollectionUtils.isEmpty(facilityInfoList)) {
            if (facilityInfoList.size() > 1) {
                Optional<FacilityInfo> facInfo = facilityInfoList.stream().filter(facilityInfo -> StringUtils.isNotBlank(facilityInfo.getFacilityStatus())
                        && StringUtils.equalsIgnoreCase(facilityInfo.getFacilityStatus(), "Active")).findAny();
                if (facInfo.isPresent()) {
                    creditFacilityStatus = facInfo.get().getFacilityStatus();
                }
            } else {
                creditFacilityStatus = facilityInfoList.get(0).getFacilityStatus();
            }
        }

        return creditFacilityStatus;
    }

    public String getFacilityStatus(List<FacilityInfo> facilityInfoList) {
        String creditFacilityStatus = "";
        if (!CollectionUtils.isEmpty(facilityInfoList)) {
            if (facilityInfoList.size() > 1) {
                Optional<FacilityInfo> facInfo = facilityInfoList.stream().filter(facilityInfo -> StringUtils.isNotBlank(facilityInfo.getFacilityStatus())
                        && StringUtils.equalsIgnoreCase(facilityInfo.getFacilityStatus(), "Active")).findAny();
                if (facInfo.isPresent()) {
                    creditFacilityStatus = facInfo.get().getFacilityStatus();
                }
            } else {
                creditFacilityStatus = facilityInfoList.get(0).getFacilityStatus();
            }
        }

        return creditFacilityStatus;
    }

    public List<FacilityInfo> getFacilityInfo(WFDSResponse wfdsResponse) {
        Gson gson = new Gson();
        List<FacilityInfo> facilityInfoList = new ArrayList<>();
        if (Objects.nonNull(wfdsResponse) && !CollectionUtils.isEmpty(wfdsResponse.getClient())) {
            wfdsResponse.getClient().forEach(client -> {
                if (!CollectionUtils.isEmpty(client.getFacility())) {
                    client.getFacility().forEach(facInfo -> {
                        if (StringUtils.isNotBlank(facInfo.getFacilityType())
                                && StringUtils.equalsIgnoreCase(facInfo.getFacilityType(), "GUIDANCE LN")
                                && StringUtils.isNotBlank(facInfo.getFacilityStatus())
                                && StringUtils.equalsIgnoreCase(facInfo.getFacilityStatus(), "Active")) {
                            log.info("Facility Info of type GuidanceLN and Active facility status: {}", gson.toJson(facInfo));
                            facilityInfoList.add(facInfo);
                        }
                    });
                }
            });
        }
        return facilityInfoList;
    }

    /**
     * Get facilityTenorValue from WFDS
     * @param ucn
     * @return
     */
    @Override
    public String getFacilityTenorValue(List<FacilityInfo> facilityList) {
        AtomicReference<String> facilityTenor = new AtomicReference<>();

        try {

            if (!CollectionUtils.isEmpty(facilityList)) {
                WFDSSnapshotResponse snapshotResponse = getFacilitySnapshot(facilityList.get(0).getFacilityId());
                if (Objects.nonNull(snapshotResponse)) {
                    snapshotResponse.getCreditFacility().forEach(facility -> {
                        if (StringUtils.isNotBlank(facility.getGesFacilityTenor())) {
                            log.info("GesFacilityTenor : {}", facility.getGesFacilityTenor());
                            facilityTenor.set(facility.getGesFacilityTenor().split(" ")[0]);
                        } else {
                            for (OutstandingType type : facility.getDimensions().getOutstandingTypes()) {
                                if (StringUtils.equalsIgnoreCase(type.getValuestring(), "COMML CD LN")) {
                                    facilityTenor.set(type.getFacilityMaximumTransactionTenorValue()
                                            .toString());
                                }
                            }
                        }
                    });
                }
            }
        } catch (Exception e) {
            log.error("Exception occurred while getting the facility tenor value : ", e);
        }

        return facilityTenor.get();
    }

    @Override
    public String getUcn(String ecid) {
        try {
            String url = cbuoUrl + "api/party/partySearch?ecid=" + ecid;
            PartySearchClientResponse partySearchList = getCBUOPartyClientResponseUnfiltered(ecid, url);
            log.info("CommonUtilityService: getClientDetailResponse >>> partySearchList : {}", partySearchList);
            List<Party> parties = partySearchList.getParties();
            for (Party party: parties) {
                if (Objects.nonNull(party.getPartyAttributes())){
                    List<AlternateIdentifiers> alternateIdentifiersList = party.getPartyAttributes().getAlternateIdentifiers();
                    if(Objects.nonNull(alternateIdentifiersList)) {
                        for (AlternateIdentifiers alternateIdentifiers : alternateIdentifiersList) {
                            if (Objects.nonNull(alternateIdentifiers) && alternateIdentifiers.getAlternatePartyIdentifierType().equalsIgnoreCase("ucn"))
                                return alternateIdentifiers.getAlternateIdentifierValue();
                        }
                    }
                }
            }
        } catch (Exception e) {
            log.info("Exception thrown when retrieving UCN from common utility service, {}", e.getMessage());
            throw new DownstreamServicesException("Error fetching UCN from common utility service", e);
        }
        return null;
    }

    private List<ClientSystemAdmin> getCBUOResponse(String ecid, String url) {

        try{
            HttpHeaders headers = new HttpHeaders();
            headers.setContentType(MediaType.APPLICATION_JSON);
            HttpEntity<InviteDetailsResponse> request = new HttpEntity<>(new InviteDetailsResponse(), headers);
            log.info("Calling CBUO Common Utility Service with endpoint URL: {} and ecid: {}", url, ecid);
            //            InviteDetailsResponse systemAdminList = restTemplate.getForObject(url, InviteDetailsResponse.class);
            ResponseEntity<InviteDetailsResponse> response = cbuoRestTemplate.exchange(url, HttpMethod.GET, request, InviteDetailsResponse.class);

            return response.getBody().getClientSystemAdmins();//systemAdminList.getClientSystemAdminList();

        } catch(Exception ex) {
            throw new DownstreamServicesException("UNEXPECTED ERROR",
                    "Received an unexpected error from CBUO Common Utility Service",
                    ex);
        }
    }

    private ArrayList getCBUOPartyClientResponse(String ecid, String url) {
        ArrayList partySearchList = new ArrayList<>();
        try{
            HttpHeaders headers = new HttpHeaders();
            headers.setContentType(MediaType.APPLICATION_JSON);
            HttpEntity<ClientSearchResponse> request = new HttpEntity<>(new ClientSearchResponse(), headers);
            log.info("Calling CBUO Common Utility Service with endpoint URL: {} and ecid: {}", url, ecid);
            ResponseEntity<ClientSearchResponse> response = cbuoRestTemplate.exchange(url, HttpMethod.GET, request, ClientSearchResponse.class);
            if(response != null && response.getBody() != null) {
                partySearchList = (ArrayList) response.getBody().getPartySearchList();
                log.info("Response from CBUO Common Utility Service >>>..  partySearchList: {}", partySearchList);
            }
        } catch(Exception ex) {
            throw new DownstreamServicesException("UNEXPECTED ERROR",
                    "Received an unexpected error from CBUO Common Utility Service",
                    ex);
        }
        return partySearchList;
    }

    private PartySearchClientResponse getCBUOPartyClientResponseUnfiltered(String ecid, String url) {
        PartySearchClientResponse partySearchList = new PartySearchClientResponse();
        try{
            HttpHeaders headers = new HttpHeaders();
            headers.setContentType(MediaType.APPLICATION_JSON);
            HttpEntity<PartySearchClientResponse> request = new HttpEntity<>(new PartySearchClientResponse(), headers);
            log.info("Calling CBUO Common Utility Service with endpoint URL: {} and ecid: {}", url, ecid);
            ResponseEntity<PartySearchClientResponse> response = cbuoRestTemplate.exchange(url, HttpMethod.GET, request, PartySearchClientResponse.class);
            if(response != null && response.getBody() != null) {
                partySearchList = response.getBody();
                log.info("Response from CBUO Common Utility Service >>>..  partySearchList: {}", partySearchList);
            }
        } catch(Exception ex) {
            throw new DownstreamServicesException("UNEXPECTED ERROR",
                    "Received an unexpected error from CBUO Common Utility Service",
                    ex);
        }
        return partySearchList;
    }

    /**
     * Get clientFacilityList from WFDS using Common Utility Service as pass through
     * @param ucn
     * @return
     */
    public WFDSResponse getClientFacilityList(String ucn) {

        HttpEntity<WFDSResponse> request = new HttpEntity<>(new WFDSResponse());
        String facilityListUrl = cbuoUrl + "api/common/getClientFacilityList?ucn=" + ucn;
        log.info("Calling CBUO Common Utility Service with endpoint URL: {} and ucn: {}", facilityListUrl, ucn);

        ResponseEntity<WFDSResponse>  wfdsResponse = null;
        try {
            wfdsResponse = cbuoRestTemplate.exchange(facilityListUrl, HttpMethod.GET, request, WFDSResponse.class);
            log.info(" facilityList successfully retrieved for ucn : {}", ucn);
        }catch (Exception ex){
            log.error("Exception while fetching facilityList : {}", ex);
        }

        return wfdsResponse.getBody();
    }

    /**
     * Get facilitySnapshot from WFDS using Common Utility Service as pass through
     * @param facilityId
     * @return
     */
    public WFDSSnapshotResponse getFacilitySnapshot(String facilityId) {
        HttpEntity<WFDSResponse> request = new HttpEntity<>(new WFDSResponse());

        String facilitySnapshotUrl = cbuoUrl + "api/common/getFacilitySnapshot?facilityId=" + facilityId;
        log.info("Calling CBUO Common Utility Service with endpoint URL: {} and facilityId: {}", facilitySnapshotUrl, facilityId);

        ResponseEntity<WFDSSnapshotResponse> snapshotResponse = null;
        try {
            snapshotResponse = cbuoRestTemplate.exchange(
                    facilitySnapshotUrl, HttpMethod.GET, request, WFDSSnapshotResponse.class);
            log.info(" facilitysnapshot successfully retrieved for facilityId: {}", facilityId);
        } catch (Exception ex){
            log.error("Exception while fetching facilityList : {}", ex);
        }

        return Objects.nonNull(snapshotResponse) ? snapshotResponse.getBody() : null;
    }

    @Override
    public String getCostCenter(String ecid) {
        try {
            String url = cbuoUrl + "api/party/partySearch?ecid=" + ecid;
            PartySearchClientResponse partySearchList = getCBUOPartyClientResponseUnfiltered(ecid, url);
            log.info("CommonUtilityService: getClientDetailResponse >>> partySearchList : {}", partySearchList);
            List<Party> parties = partySearchList.getParties();
            for (Party party: parties) {
                if (Objects.nonNull(party.getPartyAttributes())){
                    String costCenter = party.getPartyAttributes().getCostCenter();
                    if(StringUtils.isNotEmpty(costCenter)) {
                        log.info("CommonUtilityServiceImpl: cost center is {} for ecid {}", costCenter, ecid);
                        return costCenter;
                    }
                }
            }
        } catch (Exception e) {
            log.info("Exception thrown when retrieving cost center from common utility service, {}", e.getMessage());
            throw new DownstreamServicesException("Error fetching cost center from common utility service", e);
        }
        log.info("Cost center not found for ecid {}", ecid);
        return null;
    }


    public List<EmployeeDTO> getEmployeeDetails(String sid) {
        try{
            // Set the headers
            HttpHeaders headers = new HttpHeaders();
            headers.setContentType(MediaType.APPLICATION_JSON);
            List<String> stringList = Arrays.asList(sid);
            HttpEntity<List<String>> requestEntity = new HttpEntity<>(stringList, headers);
            String url = cbuoUrl + "api/common/employee-search";
            log.info("Calling CBUO Common Utility Service with endpoint URL: {} and sid: {}", url, sid);
            ResponseEntity<EmployeeSearchResponse> response = cbuoRestTemplate.exchange(url, HttpMethod.POST, requestEntity, EmployeeSearchResponse.class);
            log.info("Response from CBUO Common Utility Service >>>..  employeeSearchResponse: {}", response.getBody());
            if(Objects.nonNull(response) && response.getBody() != null) {
               List<EmployeeDTO> filteredResponse = EmployeeMapper.mapEmployeeDetailstoPSO(response.getBody());
                log.info("Response from CBUO Common Utility Service  after mapping>>>..  employeeSearchResponse: {}", filteredResponse);
                return filteredResponse;
            }
        } catch(Exception ex) {
            throw new DownstreamServicesException("UNEXPECTED ERROR",
                    "Received an unexpected error from CBUO Common Utility Service",
                    ex);
        }
        log.info("Exception while fetchning resonse from Common Utility Service for employeeSearch for sid {}", sid);

        return null;
    }

    @Override
    public ReferenceData getEntityTypeFromPC(final String referenceDataUrl) {

        return commonUtilityWebClient.get().uri(referenceDataUrl).header(CommonConstants.TRACE_ID_HEADER,
                        null != MDC.get(CommonConstants.TRACEID) ? MDC.get(CommonConstants.TRACEID) :
                                UUID.randomUUID().toString()).retrieve().onStatus(httpStatus -> !httpStatus.is2xxSuccessful(),
                        error -> Mono.error(new Exception(String.valueOf(error)))).bodyToMono(ReferenceData.class)
                .onErrorMap(ex -> new Exception("Received an unexpected error while retrieving referenceData from common utitlity service: {}", ex))
                .block();
    }

    @Override
    public String getLegalEntityDescFromPC(String legalEntityDescUrl) {

        return commonUtilityWebClient.get().uri(legalEntityDescUrl).header(CommonConstants.TRACE_ID_HEADER,
                        null != MDC.get(CommonConstants.TRACEID) ? MDC.get(CommonConstants.TRACEID) :
                                UUID.randomUUID().toString()).retrieve().onStatus(httpStatus -> !httpStatus.is2xxSuccessful(),
                        error -> Mono.error(new Exception(String.valueOf(error)))).bodyToMono(String.class)
                .onErrorMap(ex -> new Exception("Received an unexpected error while retriving legalEntityDescription from common untilty service: {}", ex))
                .block();
    }
}
