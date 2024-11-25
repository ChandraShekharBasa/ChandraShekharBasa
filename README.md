 private CmlCardContactDTO getPrimaryProgramAdmin(List<CmlCardContactDTO> contacts) {
        if (Objects.nonNull(contacts)) {
            return contacts.stream()
                    .filter(contact -> contact != null && filterPrimaryProgramAdmin(contact)).findFirst().get();
        }
    }
