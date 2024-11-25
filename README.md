private CmlCardContactDTO getPrimaryProgramAdmin(List<CmlCardContactDTO> contacts) {
    // Return null if the input list is null or empty
    if (contacts == null || contacts.isEmpty()) {
        return null;
    }

    // Find the first primary program admin in the list
    return contacts.stream()
            .filter(contact -> contact != null && filterPrimaryProgramAdmin(contact))
            .findFirst()
            .orElse(null); // Return null if no primary program admin is found
}
