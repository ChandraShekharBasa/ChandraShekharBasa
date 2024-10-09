import java.util.List;

public class ResponseBody {

    private String version;
    private Status status;
    private String error;
    private Result result;
    private String traceId;
    private String timestamp;
    private String path;

    // Getters and Setters
    public String getVersion() {
        return version;
    }

    public void setVersion(String version) {
        this.version = version;
    }

    public Status getStatus() {
        return status;
    }

    public void setStatus(Status status) {
        this.status = status;
    }

    public String getError() {
        return error;
    }

    public void setError(String error) {
        this.error = error;
    }

    public Result getResult() {
        return result;
    }

    public void setResult(Result result) {
        this.result = result;
    }

    public String getTraceId() {
        return traceId;
    }

    public void setTraceId(String traceId) {
        this.traceId = traceId;
    }

    public String getTimestamp() {
        return timestamp;
    }

    public void setTimestamp(String timestamp) {
        this.timestamp = timestamp;
    }

    public String getPath() {
        return path;
    }

    public void setPath(String path) {
        this.path = path;
    }

    // Nested static class for Status
    public static class Status {
        private int statusCode;
        private String statusMessage;

        // Getters and Setters
        public int getStatusCode() {
            return statusCode;
        }

        public void setStatusCode(int statusCode) {
            this.statusCode = statusCode;
        }

        public String getStatusMessage() {
            return statusMessage;
        }

        public void setStatusMessage(String statusMessage) {
            this.statusMessage = statusMessage;
        }
    }

    // Nested static class for Result
    public static class Result {
        private Paginate paginate;
        private List<ListItem> list;

        // Getters and Setters
        public Paginate getPaginate() {
            return paginate;
        }

        public void setPaginate(Paginate paginate) {
            this.paginate = paginate;
        }

        public List<ListItem> getList() {
            return list;
        }

        public void setList(List<ListItem> list) {
            this.list = list;
        }
    }

    // Nested static class for Paginate
    public static class Paginate {
        private int page;
        private int totalRecords;
        private int remainingRecords;

        // Getters and Setters
        public int getPage() {
            return page;
        }

        public void setPage(int page) {
            this.page = page;
        }

        public int getTotalRecords() {
            return totalRecords;
        }

        public void setTotalRecords(int totalRecords) {
            this.totalRecords = totalRecords;
        }

        public int getRemainingRecords() {
            return remainingRecords;
        }

        public void setRemainingRecords(int remainingRecords) {
            this.remainingRecords = remainingRecords;
        }
    }

    // Nested static class for ListItem
    public static class ListItem {
        private String fullName;
        private String managerName;
        private String firstName;
        private String middleName;
        private String lastName;
        private String displayedFirstName;
        private String managerSID;
        private String managerFirstName;
        private String managerLastName;
        private String managerPreferredName;
        private String managerPreferredLastName;
        private String userSID;
        private String email;
        private int phoneNo;
        private List<String> seeMe;
        private String mobileTP;
        private String personalTP;
        private String location;
        private String costCenter;
        private String costCenterName;
        private List<String> levels;
        private String buildingName;
        private String city;
        private String state;
        private String region;
        private String country;
        private String countryAbbr;
        private String timeZone;
        private String imageURL;
        private String title;
        private String preferredTitle;
        private String bankTitleCode;
        private String bankSubTitleCode;
        private int isContractor;
        private int cellphone;
        private String zoom;
        private String vanityName;
        private String positionCode;
        private String positionName;
        private String officerCode;
        private List<WorkContact> alternateWorkContacts;
        private List<WorkContact> executiveAssistants;
        private List<WorkContact> administrativeAssistants;
        private String displayedLastName;
        private boolean isRecorded;
        private String floor;
        private String taxonomyId;
        private String address;
        private String bankTitleCodeDescription;
        private String legalEntityId;
        private String legalEntityDescription;
        private String jobCodeId;
        private String jobCodeDescription;
        private String jobFamilyCode;
        private String jobFamilyDescription;
        private String lobCode;
        private String lobDescription;
        private String subLobCode;
        private String subLobDescription;

        // Getters and Setters (for all fields)
        // Example:
        public String getFullName() {
            return fullName;
        }

        public void setFullName(String fullName) {
            this.fullName = fullName;
        }

        // All other getters and setters...
        // (can be auto-generated with your IDE)
    }

    // Nested static class for WorkContact
    public static class WorkContact {
        private String SID;

        // Getters and Setters
        public String getSID() {
            return SID;
        }

        public void setSID(String SID) {
            this.SID = SID;
        }
    }
}
