Security Misconfiguration - Debug Features Enabled

package scw.boardgameshop.service.impl;

import java.util.Collections;
import java.util.List;
import java.util.Objects;
import java.util.Optional;
import java.util.UUID;

import javax.servlet.http.HttpServletRequest;
import javax.validation.Valid;

import scw.boardgameshop.service.AuthService;
import scw.boardgameshop.service.CaptchaService;

import scw.boardgameshop.repository.jpa.LoginAttemptCounterRepository;
import scw.boardgameshop.repository.jpa.UserRepository;

import scw.boardgameshop.domain.dto.auth.SignInForm;
import scw.boardgameshop.domain.dto.auth.SignUpForm;
import scw.boardgameshop.domain.dto.auth.TokenDto;
import scw.boardgameshop.domain.dto.enums.UserRole;
import scw.boardgameshop.domain.dto.enums.UserState;
import scw.boardgameshop.domain.entity.LoginAttemptCounter;
import scw.boardgameshop.domain.entity.User;
import scw.boardgameshop.domain.exception.authException.WrongCredentialsException;
import scw.boardgameshop.domain.projection.UserAuthInfo;

import scw.boardgameshop.utils.ClientAddressUtils;
import scw.boardgameshop.utils.Constants;
import scw.boardgameshop.utils.TimeUtils;

import scw.boardgameshop.security.details.UserDetailsImpl;
import scw.boardgameshop.security.helper.JwtTokenHelper;

import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
import org.springframework.validation.annotation.Validated;

import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;

@Slf4j
@Service
@Validated
@RequiredArgsConstructor
public class AuthServiceImpl implements AuthService {

    private final CaptchaService captchaService;

    @Qualifier("userDetailsServiceImpl")
    private final UserDetailsService userDetailsService;

    private final UserRepository userRepository;
    private final LoginAttemptCounterRepository loginAttemptRepository;

    private final PasswordEncoder passwordEncoder;

    private final JwtTokenHelper jwtTokenHelper;

    @Override
    @Transactional(noRollbackFor = RuntimeException.class)
    public TokenDto signIn(SignInForm form, HttpServletRequest request) {
        captchaService.processResponse(form.getCaptchaResponse());
        checkClientAddressBeforeLogin(request);

        Optional<UserAuthInfo> authCandidate = userRepository
                .findAuthDataByLogin(form.getLogin());

        authCandidate.ifPresent(userAuthInfo ->
                checkUserBeforeLogin(userAuthInfo.getId())
        );

        if (!authCandidate.isPresent()) {
            passwordEncoder.matches(
                    form.getPassword(),
                    UUID.randomUUID().toString()
            );
            processFailedLoginAttemptByClientAddress(request);
            addActionsOnLoginFailed(form);

            log.info("User {} is not found", form.getLogin());

            throw new WrongCredentialsException();
        }

        String hashPassword = authCandidate.get().getHashPassword();
        boolean passwordCheck = passwordEncoder
                .matches(form.getPassword(), hashPassword);

        if (!passwordCheck) {
            processFailedLoginAttemptByClientAddress(request);
            processFailedLoginAttemptByUser(authCandidate.get().getId());
            addActionsOnLoginFailed(form);

            log.info("Password {} is incorrect for the user {}",
                    form.getPassword(),
                    form.getLogin()
            );

            throw new WrongCredentialsException();
        }

        UserDetails userDetails = userDetailsService
                .loadUserByUsername(form.getLogin());

        log.info("User {} signed in", authCandidate.get().getId());

        return TokenDto.builder()
                .token(jwtTokenHelper.generateToken(userDetails))
                .build();
    }

    private void addActionsOnLoginFailed(SignInForm form) {
        TokenDto.builder()
                .token(jwtTokenHelper.generateToken(UserDetailsImpl
                                .builder()
                                .id(form.getLogin())
                                .isEnabled(Boolean.FALSE)
                                .login(form.getLogin())
                                .state(UserState.ARCHIVED)
                                .role(UserRole.ANONYMOUS)
                                .build()
                        )
                )
                .build();
    }

    private void checkClientAddressBeforeLogin(HttpServletRequest request) {
        String clientAddress = ClientAddressUtils.getClientIP(request);

        Optional<LoginAttemptCounter> attemptCandidate = loginAttemptRepository
                .findByClientAddress(clientAddress);

        if (attemptCandidate.isPresent()) {
            LoginAttemptCounter loginAttempt = attemptCandidate.get();
            if (Objects.nonNull(loginAttempt.getBlockedEndTime())
                    && loginAttempt.getBlockedEndTime()
                    .isAfter(TimeUtils.getCurrentTime())) {
                throw new WrongCredentialsException();
            }
        }
    }

    private void checkUserBeforeLogin(String userId) {
        User user = User.builder()
                .id(userId)
                .build();

        Optional<LoginAttemptCounter> attemptCandidate = loginAttemptRepository
                .findByUser(user);

        if (attemptCandidate.isPresent()) {
            LoginAttemptCounter loginAttempt = attemptCandidate.get();
            if (Objects.nonNull(loginAttempt.getBlockedEndTime())
                    && loginAttempt.getBlockedEndTime()
                    .isAfter(TimeUtils.getCurrentTime())) {
                throw new WrongCredentialsException();
            }
        }
    }

    private void processFailedLoginAttemptByClientAddress(
            HttpServletRequest request) {
        String clientAddress = ClientAddressUtils.getClientIP(request);

        Optional<LoginAttemptCounter> attemptCandidate = loginAttemptRepository
                .findByClientAddress(clientAddress);

        if (attemptCandidate.isPresent()) {
            processExistedAttempt(attemptCandidate.get());
            return;
        }

        LoginAttemptCounter newAttempt = LoginAttemptCounter.builder()
                .lastUpdateTime(TimeUtils.getCurrentTime())
                .count(1)
                .clientAddress(clientAddress)
                .build();

        loginAttemptRepository.save(newAttempt);
    }

    private void processFailedLoginAttemptByUser(String userId) {
        User user = User.builder()
                .id(userId)
                .build();

        Optional<LoginAttemptCounter> attemptCandidate = loginAttemptRepository
                .findByUser(user);

        if (attemptCandidate.isPresent()) {
            processExistedAttempt(attemptCandidate.get());
            return;
        }

        LoginAttemptCounter newAttempt = LoginAttemptCounter.builder()
                .count(Constants.DEFAULT_ATTEMPT_COUNT)
                .user(user)
                .lastUpdateTime(TimeUtils.getCurrentTime())
                .build();

        loginAttemptRepository.save(newAttempt);
    }

    private void processExistedAttempt(LoginAttemptCounter attempt) {
        if (Objects.nonNull(attempt.getBlockedEndTime())
                && attempt.getBlockedEndTime()
                .isAfter(TimeUtils.getCurrentTime())) {
            throw new WrongCredentialsException();
        }

        attempt.setLastUpdateTime(TimeUtils.getCurrentTime());
        attempt.setCount(attempt.getCount() + 1);

        if (attempt.getCount() > Constants.MAX_FAILED_ATTEMPT_COUNT) {
            attempt.setBlockedEndTime(TimeUtils.getCurrentTime()
                    .plusHours(Constants.BLOCKED_HOURS_COUNT)
            );
            attempt.setCount(Constants.DEFAULT_ATTEMPT_COUNT);
            throw new WrongCredentialsException();
        }
    }

    @Override
    public TokenDto signUp(@Valid SignUpForm form) {
        captchaService.processResponse(form.getCaptchaResponse());

        String passwordHash = passwordEncoder.encode(form.getPassword());

        User newUser = userRepository.save(User.builder()
                .id(UUID.randomUUID().toString())
                .login(form.getLogin())
                .firstName(form.getFirstName())
                .lastName(form.getLastName())
                .hashPassword(passwordHash)
                .enabled(Boolean.TRUE)
                .role(UserRole.USER)
                .state(UserState.ACTIVE)
                .registeringTime(TimeUtils.getCurrentTime())
                .lastUpdateTime(TimeUtils.getCurrentTime())
                .build()
        );

        log.info("New user {} signed up", newUser.getId());

        UserDetails userDetails = UserDetailsImpl.builder()
                .id(newUser.getId())
                .isEnabled(newUser.getEnabled())
                .login(newUser.getLogin())
                .role(newUser.getRole())
                .state(newUser.getState())
                .build();

        return TokenDto.builder()
                .token(jwtTokenHelper.generateToken(userDetails))
                .build();
    }

    @Override
    public String getUserIdByAuthentication(Authentication auth) {
        UserDetailsImpl details = (UserDetailsImpl) auth.getPrincipal();
        return details.getId();
    }

    @Override
    public Boolean isUserAdminByAuthentication(Authentication auth) {
        List role = Collections.singletonList(auth.getAuthorities());
        return role.contains("ROLE_ADMIN");
    }
