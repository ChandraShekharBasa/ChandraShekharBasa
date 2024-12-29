  @GetMapping(value = "/search-by-patient-result")
    public String getAnalysisSearchResultByPatient(Model model,
            @RequestParam(required = false, defaultValue = "") String patient) {
        model.addAttribute("filter", StringEscapeUtils.escapeHtml(patient));
        model.addAttribute("results", analysisService.getAnalyzesByPatient(patient));
        return "analysis-requests-result";
    }
