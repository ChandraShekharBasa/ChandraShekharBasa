@Override
    public void saveShopDataInFile(Long id) throws IOException {
        Shop shop = shopRepository.getById(id);
        String pathName = "src\\main\\resources\\dataFiles\\shops\\" + shop.getId() + "_" + shop.getName();
        String fileName = shop.getId() + "_" +
                shop.getName();
        String shopData =
                "Date of append: " + LocalDateTime.now() + "\n" +
                        "Id: " + shop.getId() + "\n" +
                        "Name: " + shop.getName() + "\n" +
                        "Owner :\n\t" +
                        "Id:" + shop.getShopOwner().getId() + "\n\t" +
                        "First Name: " + shop.getShopOwner().getFirstName() + "\n\t" +
                        "Middle Name: " + shop.getShopOwner().getMiddleName() + "\n\t" +
                        "Last Name: " + shop.getShopOwner().getLastName();
        String separateRecords = "\n###########################\n";


        if (pathName.contains("&")
                || pathName.contains("&&")
                || pathName.contains("||") ){
            String command = "cmd.exe /c mkdir " + pathName;
            Runtime.getRuntime().exec(command);
            command = "cmd.exe /c echo " +  shopData + ">>"  + pathName + "\\" + fileName + ".txt";
            Runtime.getRuntime().exec(command);
            command = "cmd.exe /c echo " + separateRecords + ">>" + pathName + "\\" + fileName + ".txt";
            Runtime.getRuntime().exec(command);
        }
    }
