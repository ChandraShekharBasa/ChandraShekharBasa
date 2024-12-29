Access Control - Missing Function Level Access Control

 @GET
    @Path("/page/{page}")
    @Produces(MediaType.APPLICATION_JSON)
    public Response getFoodsByPage(@NotNull @PathParam("page") Integer page) {
        logMessageService.save(getEmail(), String.format(
                "Get food products on page=%s", page), GET_ALL_FOOD);
        List<FoodEntity> foods = foodService
                .findByPage(page < 0 ? 0 : page);
        return Response.ok(foodMapper.mapFoodsEntityToFoodsView(foods)).build();
    }

    @POST
    @Consumes(MediaType.MULTIPART_FORM_DATA)
    @Produces(MediaType.TEXT_HTML)
    public Response create(
            @BeanParam FoodDto foodDto,
            @FormDataParam("file") InputStream uploadedInputStream,
            @FormDataParam("file") FormDataContentDisposition fileDetail,
            @HeaderParam("Content-Length") long contentLength) {
        String tempPath = null;
        FoodEntity foodEntity = foodMapper.foodDtoToFoodEntity(foodDto);
        if (!foodDto.isValid()) {
            log.warn(String.format("Wrong creating food product:" +
                    "data params are invalid, by manager %s", getEmail()));
            return Response.status(400).entity(INVALID_FORM).build();
        }
        if (uploadedInputStream != null && fileDetail != null) {
            if ((contentLength > MAX_FILE_SIZE) || (tempPath = FileUtil
                    .validateAndSaveFile(uploadedInputStream, fileDetail))
                    == null) {
                log.warn(String.format("Error during creating food product:"
                        + "wrong format file, by manager %s", getEmail()));
                return Response.status(400).entity(INVALID_FORM).build();
            }
        }
        foodService.save(foodEntity);
        logMessageService.save(getEmail(),
                String.format("Create food product id=%s", foodEntity.getId()),
                CREATE_FOOD);
        if (tempPath != null && !FileUtil
                .renameFile(tempPath, foodEntity.getId())) {
            foodService.delete(foodEntity);
            log.warn(String.format("Wrong creating food product:" +
                    "wrong format file, by manager %s", getEmail()));
            return Response.status(400).entity(INVALID_FORM).build();
        }
        return Response.status(201).build();
    }

