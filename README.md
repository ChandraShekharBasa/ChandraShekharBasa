import static org.mockito.Mockito.*;
import static org.junit.jupiter.api.Assertions.*;

import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;
import org.slf4j.Logger;
import java.util.List;
import java.util.Collections;

@ExtendWith(MockitoExtension.class)
class YourServiceTest {

    @InjectMocks
    private YourService yourService;

    @Mock
    private CMLCardProductService cmlCardProductService;

    @Mock
    private KYCInformationUtilityService kycInformationUtilityService;

    @Mock
    private DocManagementService docManagementService;

    @Mock
    private OrchestrationService orchestrationService;

    @Mock
    private EmailService emailService;

    @Mock
    private Logger appLogger;

    @Mock
    private KYCInformationEntity kycInformationEntity;

    @Mock
    private DocumentMetaDataEntity documentMetaDataEntity;

    @Mock
    private Product product;

    private final Long kycId = 123L;

    @BeforeEach
    void setUp() {
        when(product.getType()).thenReturn(COLTConstants.ACCELERATOR_CARD);
    }

    @Test
    void testSubmitContract_Success() {
        // Arrange
        when(cmlCardProductService.getProductsForKYC(kycId)).thenReturn(Collections.singletonList(product));
        when(kycInformationUtilityService.getKycInformationEntity(kycId)).thenReturn(kycInformationEntity);
        when(docManagementService.getCCADocForAcceleratorCard(kycId)).thenReturn(documentMetaDataEntity);
        when(orchestrationService.submitContract(any(ContractBaseRequest.class))).thenReturn(CommonConstants.CONTRACT_API_SUBMIT_SUCCESS);

        // Act
        yourService.submitContract(kycId);

        // Assert
        verify(orchestrationService).submitContract(any(ContractBaseRequest.class));
        verify(kycInformationUtilityService).saveKycInformation(kycInformationEntity);
        verify(appLogger).info("Updating KycInformation with contract submission status: {} for kycId {}",
                CommonConstants.CONTRACT_API_SUBMIT_SUCCESS, kycId);
        assertEquals(CommonConstants.CONTRACT_API_SUBMIT_SUCCESS, kycInformationEntity.getContractSubmitStatus());
    }

    @Test
    void testSubmitContract_Failure() {
        // Arrange
        when(cmlCardProductService.getProductsForKYC(kycId)).thenReturn(Collections.singletonList(product));
        when(kycInformationUtilityService.getKycInformationEntity(kycId)).thenReturn(kycInformationEntity);
        when(docManagementService.getCCADocForAcceleratorCard(kycId)).thenReturn(documentMetaDataEntity);
        when(orchestrationService.submitContract(any(ContractBaseRequest.class))).thenThrow(new DownstreamServicesException());

        // Act & Assert
        DownstreamServicesException exception = assertThrows(DownstreamServicesException.class, () -> {
            yourService.submitContract(kycId);
        });

        // Verifications
        verify(emailService).sendEmailForServiceFailure(kycId, CommonConstants.CONTRACT_API);
        verify(kycInformationUtilityService).saveKycInformation(kycInformationEntity);
        assertEquals(CommonConstants.CONTRACT_API_SUBMIT_FAILED, kycInformationEntity.getContractSubmitStatus());
    }

    @Test
    void testSubmitContract_ProductsListIsNull() {
        // Arrange
        when(cmlCardProductService.getProductsForKYC(kycId)).thenReturn(null);

        // Act
        yourService.submitContract(kycId);

        // Assert
        verify(orchestrationService, never()).submitContract(any(ContractBaseRequest.class));
        verify(emailService, never()).sendEmailForServiceFailure(anyLong(), anyString());
    }
}
