import static org.mockito.Mockito.*;
import static org.junit.jupiter.api.Assertions.*;

import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.MockitoAnnotations;

public class CmlCardPaymentInfoServiceImplTest {

    @Mock
    private KycPaymentInfoRepository kycPaymentInfoRepository;

    @InjectMocks
    private CmlCardPaymentInfoServiceImpl cmlCardPaymentInfoServiceImpl;

    @BeforeEach
    public void setup() {
        MockitoAnnotations.initMocks(this);
    }

    @Test
    public void testGetRebateAccountInfo_whenExceptionThrown_shouldThrowOnboardingDatabaseException() {
        String kycId = "12345";

        // Mock repository to throw an exception
        when(kycPaymentInfoRepository.findRebateDetailsForKycId(Long.parseLong(kycId)))
                .thenThrow(new RuntimeException("Database error"));

        // Verify that the exception is thrown
        OnboardingDatabaseException exception = assertThrows(OnboardingDatabaseException.class, () -> {
            cmlCardPaymentInfoServiceImpl.getRebateAccountInfo(kycId);
        });

        // Verify the exception message
        assertEquals("An exception occurred trying to get payment info with kycId: " + kycId, exception.getMessage());

        // Verify that the error code was set in MDC
        assertEquals("DB_ERROR", exception.getErrorCode());
    }
}



Employee Search request: EmployeeSearch(query=Query(impersonate=null, criteria=null, bulk=Bulk(field=_all, values=[basa])))
{
  "query": {
    "impersonate": null,
    "criteria": null,
    "bulk": {
      "field": "_all",
      "values": ["basa"]
    }
  }
}
