if(fileService.uploadFile(file, principalUser.getUser())) {
   // The file path or content could be passed to the fileService method for storage
}

if (fileService.serializationUploadFile(file, principalUser.getUser())) {
   // Again, the file path/content could be passed to a system-level operation
}

if (fileService.downLoadFile(selectedFile, chosenDir)) {
   // The file paths (selectedFile, chosenDir) may be used in an OS command later
}

if (fileService.deleteFile(nameOfDeletingFile, principalUser.getUser())) {
   // If nameOfDeletingFile is passed to any system-level command, this is another place to check for vulnerabilities
}
