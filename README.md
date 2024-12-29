package com.scw.directoryService.controller;

import com.scw.directoryService.ApplicationMain;
import com.scw.directoryService.security.Principal;
import com.scw.directoryService.service.FileService;
import javafx.beans.property.SimpleStringProperty;
import javafx.collections.FXCollections;
import javafx.collections.ObservableList;
import javafx.fxml.FXML;
import javafx.scene.control.*;
import javafx.stage.DirectoryChooser;
import javafx.stage.FileChooser;
import javafx.stage.Stage;

import java.io.File;
import java.util.logging.Logger;

public class FileFolderController implements CommonController {

    private static final Logger logger = Logger.getLogger(FileFolderController.class.getName());

    private Stage stage;
    private Principal principalUser;
    private FileService fileService;

    public void setPrincipal(Principal principalUser){this.principalUser = principalUser;}
    public void setFileService(FileService fileService) {
        this.fileService = fileService;
    }

    private ObservableList<File> reportsList;


    @Override
    public void setStage(Stage stage) {
        this.stage = stage;
    }

    @FXML
    private Label correctMessageLabel;
    @FXML
    private Label incorrectMessageLabel;
    @FXML
    public TextField fileNameField;
    @FXML
    public Button uploadObjectReportButton;
    @FXML
    public Label notificationAboutBinFilesLabel;

    @FXML
    private TableView<File> tableWithFiles;
    @FXML
    private TableColumn<File, String> fileName;
    @FXML
    private TableColumn<File, String> fileSize;
    
    @FXML
    public void initialize() {
        tableWithFiles.getSelectionModel().selectedItemProperty().addListener(
            (observable, oldValue, newValue) -> showListOfFiles());

        fileName.setCellValueFactory(cellData
            -> new SimpleStringProperty(cellData.getValue().getName()));
        fileSize.setCellValueFactory(cellData
            -> new SimpleStringProperty(String.valueOf(cellData.getValue().length())));
    }

    public boolean uploadReport() {
        correctMessageLabel.setText("");
        incorrectMessageLabel.setText("");

        String userLogin = principalUser.getUser().getLogin();
        String departmentName = principalUser.getUser().getDepartment().name();
        File file = openFileFromDisk();

        if(fileService.uploadFile(file, principalUser.getUser())) {

            reportsList = FXCollections.observableArrayList();
            reportsList.addAll(fileService.getFilesForDepartment(departmentName));
            tableWithFiles.setItems(reportsList);

            logger.info("user " + userLogin +
                    " successful upload file for department " + departmentName);

            correctMessageLabel.setText("file was uploaded correctly");
            return true;
        }

        incorrectMessageLabel.setText("dear user " + userLogin +
            " file isn't uploaded;" +
            " check file name (should be unique);" +
            " check file size (must be less than 5 mb);" +
            " check file extension (must be .jpeg or .xml)");

        return false;
    }

    @FXML
    public boolean uploadReportAsObject() {
        String departmentName = principalUser.getUser().getDepartment().name();
        String userLogin = principalUser.getUser().getLogin();
        File file = openFileFromDisk();

        if (fileService.serializationUploadFile(file, principalUser.getUser())){
            logger.info("manager " + userLogin +
                " successfully uploaded file using serialization ");

            reportsList = FXCollections.observableArrayList();
            reportsList.addAll(fileService.getFilesForDepartment(departmentName));
            tableWithFiles.setItems(reportsList);

            correctMessageLabel.setText("you successfully upload file from .bin");
            return true;
        }
        logger.warning("manager " + userLogin +
            " trying upload .bin file;" +
            " the operation was performed incorrectly; " +
            " there may be an attempt to load an dubious serialized file");
        incorrectMessageLabel.setText("upload file from .bin was performed incorrectly; " +
            "please, try again, or please contact to your administrator");
        return false;
    }

    @FXML
    private boolean downloadReport(){
        incorrectMessageLabel.setText("");
        String userLogin = principalUser.getUser().getLogin();
        File selectedFile = tableWithFiles.getSelectionModel().getSelectedItem();

        if (selectedFile != null) {
            DirectoryChooser dirChooser = new DirectoryChooser();
            File chosenDir = dirChooser.showDialog(stage);

            if(fileService.downLoadFile(selectedFile, chosenDir)){
                logger.info("download file was performed correctly " +
                    "for user " + userLogin);
                return true;
            }
        }
        incorrectMessageLabel.setText("download file was performed incorrectly; " +
            "please, try again, or please contact to your administrator");
        return false;
    }

    @FXML
    private boolean deleteReport() {
        correctMessageLabel.setText("");
        incorrectMessageLabel.setText("");

        String nameOfDeletingFile = fileNameField.getText();
        String usersDepartment = principalUser.getUser().getDepartment().name();
        String userLogin = principalUser.getUser().getLogin();

        if (fileService.deleteFile(nameOfDeletingFile, principalUser.getUser())){
            reportsList = FXCollections.observableArrayList();
            reportsList.addAll(fileService.getFilesForDepartment(usersDepartment));
            tableWithFiles.setItems(reportsList);

            fileNameField.setText("");

            logger.info("the delete file operation was performed correctly " +
                "for user " + userLogin);
            correctMessageLabel.setText("you successfully deleted the file");
            return true;
        }
        logger.warning("the delete file operation was performed incorrectly " +
            "for user " + userLogin);
        incorrectMessageLabel.setText("the delete file operation was performed incorrectly; " +
            "please, try again, or please contact to your administrator");
        return false;

    }

    @Override
    public void prepare() {
        String departmentName = principalUser.getUser().getDepartment().name();
        String roleOfUser = principalUser.getUser().getRole().name();

        stage.setTitle(departmentName + " file folder");
        stage.setWidth(1200d);
        stage.setHeight(1200d);
        notificationAboutBinFilesLabel.setText("only managers can upload a .bin file");
        correctMessageLabel.setText("");
        incorrectMessageLabel.setText("");

        showListOfFiles();
        reportsList = FXCollections.observableArrayList();
        reportsList.addAll(fileService.getFilesForDepartment(departmentName));
        tableWithFiles.setItems(reportsList);

        if(roleOfUser.equals("ADMIN") || roleOfUser.equals("USER"))
            uploadObjectReportButton.setDisable(true);
    }
    
    private void showListOfFiles() {
        String departmentName = principalUser.getUser().getDepartment().name();
        fileService.getFilesForDepartment(departmentName);
    }

    public void backToPreviousPage() {
        if(principalUser.getUser().getRole().name().equals("USER"))
            ApplicationMain.setRoot("usersPage");

        ApplicationMain.setRoot("adminPage");
    }

    private File openFileFromDisk() {
        return new FileChooser().showOpenDialog(stage);
    }

}

