public class RequestBody {

    private Query query;

    // Getters and Setters
    public Query getQuery() {
        return query;
    }

    public void setQuery(Query query) {
        this.query = query;
    }

    // Nested static class for Query
    public static class Query {
        private String impersonate;
        private Criteria criteria;
        private Bulk bulk;

        // Getters and Setters
        public String getImpersonate() {
            return impersonate;
        }

        public void setImpersonate(String impersonate) {
            this.impersonate = impersonate;
        }

        public Criteria getCriteria() {
            return criteria;
        }

        public void setCriteria(Criteria criteria) {
            this.criteria = criteria;
        }

        public Bulk getBulk() {
            return bulk;
        }

        public void setBulk(Bulk bulk) {
            this.bulk = bulk;
        }
    }

    // Nested static class for Criteria
    public static class Criteria {
        private String field;
        private String value;

        // Getters and Setters
        public String getField() {
            return field;
        }

        public void setField(String field) {
            this.field = field;
        }

        public String getValue() {
            return value;
        }

        public void setValue(String value) {
            this.value = value;
        }
    }

    // Nested static class for Bulk
    public static class Bulk {
        private String field;
        private List<String> values;

        // Getters and Setters
        public String getField() {
            return field;
        }

        public void setField(String field) {
            this.field = field;
        }

        public List<String> getValues() {
            return values;
        }

        public void setValues(List<String> values) {
            this.values = values;
        }
    }
}

