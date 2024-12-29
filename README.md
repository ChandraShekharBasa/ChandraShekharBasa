Accessibility information
This page has four areas: A sidebar with information about the challenge, a tree view “File Explorer” containing the project files, a “Code blocks” section with a list of code blocks relevant to the challenge, and a code viewer section. The code viewer section has two groups of toggle buttons at the top, to select and compare the vulnerable code with possible solutions. One group selects the code comparison base and the other selects a solution to submit as an answer. Below those are tabs displaying opened files, with the currently opened file presented in a table. The presentation of the code viewer table depends on the ‘inline diff view’ option under editor settings. If unchecked, the table has 5 columns: Code block indicator, two for line number and code of the selected comparison base, and two for the line number and code of the selected solution variation. If ‘inline diff’ is checked, the table has 4 columns: Code block indicator, line number for the code from the comparison base, line number for the code from the selected solution, and a column with the code itself. In both presentations the code block indicator is only present in the first line of a code block and only if ‘vulnerable code’ is the current comparison base. Each line of code within a code block is prefixed with ‘CBX’, where X is a number used to distinguish between code blocks. Deleted code lines are prefixed with a ‘-’ symbol, and added lines with a ‘+’ symbol. Opening category information, pressing the hints or submit buttons will open a modal dialog.

Skip to Code Editor
Identify the vulnerability
Fix the vulnerability
Fix the vulnerability
Find and select the solution that fixes the vulnerability listed below. Each solution may take a different approach to address the problem, but only one solution is correct.

Vulnerability Category
Access Control - Missing Function Level Access Control

Actions
Attempts left: 1

View shortcuts keyboard hotkey:?

File Explorer
Highlighted files only: 224 files hidden


gateway-service1 code blocks
src1 code blocks
main1 code blocks
java1 code blocks
com1 code blocks
csw1 code blocks
cybersport1 code blocks
gateway1 code blocks
rest1 code blocks
*PlayerRestService.java1 code blocks
1
Code blocks
PlayerRestService.java:89-89
Comparison baseComparison base 
Selected solutionSelected solution 
package com.csw.cybersport.gateway.rest;

import com.csw.cybersport.common.exception.JWTException;
import com.csw.cybersport.gateway.converter.PlayerConverter;
import com.csw.cybersport.gateway.exception.*;
import com.csw.cybersport.gateway.persistence.dto.PageRequestDto;
import com.csw.cybersport.gateway.persistence.dto.PlayerDto;
import com.csw.cybersport.gateway.persistence.dto.UpdatePlayerDto;
import com.csw.cybersport.gateway.proto.generated.PlayerProto.PlayerInfo;
import com.csw.cybersport.gateway.proto.generated.PlayerProto.PlayerInfoList;
import com.csw.cybersport.gateway.service.LogService;
import com.csw.cybersport.gateway.service.PlayerService;
import com.csw.cybersport.gateway.util.RESTUtil;

import javax.annotation.security.PermitAll;
import javax.annotation.security.RolesAllowed;
import javax.inject.Inject;
import javax.jms.BytesMessage;
import javax.jms.JMSException;
import javax.validation.Valid;
import javax.ws.rs.*;
import javax.ws.rs.core.MediaType;
import javax.ws.rs.core.Response;
import java.util.List;
import java.util.Optional;

import static com.csw.cybersport.common.constants.JMSConstants.RESPONSE_MESS_PROPERTY;
import static com.csw.cybersport.gateway.util.Constants.*;
import static com.csw.cybersport.common.constants.JMSConstants
        .RESPONSE_STATUS_PROPERTY;

@Path("/players")
public class PlayerRestService {

    @Inject
    private LogService logService;

    @Inject
    private PlayerService playerService;

    @Inject
    private PlayerConverter playerConverter;

    /**
     * Getting player info by id.
     * Method is available for user, manager, admin.
     *
     * @param playerId
     * @return Response
     * @throws JMSException
     * @throws GatewayTimeoutException
     * @throws BadGatewayException
     * @throws JWTException
     * @throws SessionException
     * @throws CryptoException
     */
    @GET
    @Path("/{playerId}")
    @Produces(MediaType.APPLICATION_JSON)
    @RolesAllowed({USER, MANAGER, ADMIN})
    public Response getPlayerById(@PathParam("playerId") Long playerId)
            throws JMSException, GatewayTimeoutException, BadGatewayException,
            JWTException, SessionException, CryptoException {
        BytesMessage bytesMessage = playerService
                .getById(playerId);
        String status = bytesMessage
                .getStringProperty(RESPONSE_STATUS_PROPERTY);
        PlayerInfo playerInfo = playerService
                .getPlayerInfoFromByteMessage(bytesMessage);
        PlayerDto playerDto = playerConverter.decrypt(playerInfo);
        return RESTUtil.getResponseStatus(status, playerDto);
    }

    /**
     * Getting a list of players.
     * Available to admin.
     *
     * @param queryPage
     * @return
     * @throws JMSException
     * @throws GatewayTimeoutException
     * @throws BadGatewayException
     * @throws JWTException
     * @throws SessionException
     * @throws CryptoException
     */
    @GET
    @Produces(MediaType.APPLICATION_JSON)
    @PermitAll
    public Response getPlayers(@QueryParam("page") Integer queryPage)
            throws JMSException, GatewayTimeoutException, BadGatewayException,
            JWTException, SessionException, CryptoException {
        Integer page = Optional.ofNullable(queryPage).orElse(0);
        PageRequestDto pageRequest =
                new PageRequestDto(page, PageRequestDto.MAX_ELEMENTS_PER_PAGE);
        BytesMessage bytesMessage = playerService.getAll(pageRequest);
        String status = bytesMessage
                .getStringProperty(RESPONSE_STATUS_PROPERTY);
        PlayerInfoList playerInfoList = playerService
                .getPlayerInfoListFromByteMessage(bytesMessage);
        List<PlayerDto> playerDtoList =
                playerConverter.decrypt(playerInfoList);
        return RESTUtil.getResponseStatus(status, playerDtoList);
    }

    /**
     * Creating player info
     * Method is available for self
     *
     * @param playerDto
     * @return
     * @throws JMSException
     * @throws GatewayTimeoutException
     * @throws BadGatewayException
     * @throws JWTException
     * @throws SessionException
     * @throws CryptoException
     */
    @POST
    @Consumes(MediaType.APPLICATION_FORM_URLENCODED)
    @RolesAllowed({USER})
    public Response createPlayerInfo(@Valid @BeanParam PlayerDto playerDto)
            throws JMSException, GatewayTimeoutException, BadGatewayException,
            JWTException, SessionException, CryptoException {
        logService.info("Creating player " + playerDto);
        PlayerInfo playerInfo = playerConverter.encrypt(
                playerDto
        );

        BytesMessage bytesMessage = playerService.save(playerInfo);
        String status = bytesMessage
                .getStringProperty(RESPONSE_STATUS_PROPERTY);
        String message = bytesMessage
                .getStringProperty(RESPONSE_MESS_PROPERTY);
        return RESTUtil.getResponseStatus(status, message);
    }

    /**
     * Remove a player info (players can only be deleted by the owner user)
     * Method is available for self
     *
     * @return
     * @throws JMSException
     * @throws GatewayTimeoutException
     * @throws BadGatewayException
     * @throws JWTException
     * @throws SessionException
     * @throws CryptoException
     */
    @DELETE
    @Produces(MediaType.APPLICATION_JSON)
    @RolesAllowed({USER})
    public Response deletePlayer()
            throws JMSException, GatewayTimeoutException, BadGatewayException,
            JWTException, SessionException, CryptoException {
        BytesMessage bytesMessage = playerService.delete();
        String status = bytesMessage
                .getStringProperty(RESPONSE_STATUS_PROPERTY);
        return RESTUtil.getResponseStatus(status);
    }

    /**
     *
     * Update player info
     * Method is available for self
     *
     * @param playerDto
     * @return
     * @throws JMSException
     * @throws GatewayTimeoutException
     * @throws BadGatewayException
     * @throws JWTException
     * @throws SessionException
     * @throws CryptoException
     */
    @PUT
    @Consumes(MediaType.APPLICATION_FORM_URLENCODED)
    @RolesAllowed({USER})
    public Response updatePlayer(@Valid @BeanParam PlayerDto playerDto)
            throws JMSException, GatewayTimeoutException, BadGatewayException,
            JWTException, SessionException, CryptoException {
        BytesMessage bytesMessage = playerService.updatePlayer(
                playerConverter.encrypt(playerDto));
        String status = bytesMessage
                .getStringProperty(RESPONSE_STATUS_PROPERTY);
        String message = bytesMessage
                .getStringProperty(RESPONSE_MESS_PROPERTY);
        return RESTUtil.getResponseStatus(status, message);
    }

    /**
     *
     * adding a player to a team
     * Method is available for self
     *
     * @param teamId
     * @return
     * @throws JMSException
     * @throws GatewayTimeoutException
     * @throws BadGatewayException
     * @throws JWTException
     * @throws SessionException
     * @throws CryptoException
     */
    @PUT
    @Path("/add-to-team/{teamId}")
    @Consumes(MediaType.APPLICATION_FORM_URLENCODED)
    @RolesAllowed({USER})
    public Response updatePlayerTeam(@PathParam("teamId") Long teamId)
            throws JMSException, GatewayTimeoutException, BadGatewayException,
            JWTException, SessionException, CryptoException {
        BytesMessage bytesMessage = playerService.addToTeam(teamId);
        String status = bytesMessage
                .getStringProperty(RESPONSE_STATUS_PROPERTY);
        return RESTUtil.getResponseStatus(status);
    }

    /**
     * Updating player info
     * Method is available for admin
     *
     * @param playerId
     * @param updatePlayerDto
     * @return
     * @throws JMSException
     * @throws GatewayTimeoutException
     * @throws BadGatewayException
     * @throws JWTException
     * @throws SessionException
     * @throws CryptoException
     */
    @PUT
    @Path("/{id}")
    @Consumes(MediaType.APPLICATION_FORM_URLENCODED)
    @RolesAllowed({ADMIN})
    public Response updateShortPlayerInfo(
            @PathParam("id") Long playerId,
            @Valid @BeanParam UpdatePlayerDto updatePlayerDto)
            throws JMSException, GatewayTimeoutException, BadGatewayException,
            JWTException, SessionException, CryptoException {
        BytesMessage bytesMessage = playerService
                .updateShortPlayerInfo(playerId, updatePlayerDto);
        String status = bytesMessage
                .getStringProperty(RESPONSE_STATUS_PROPERTY);
        return RESTUtil.getResponseStatus(status);
    }

}


Access Control - Missing Function Level Access Control
