# ASP.NET Security Best Practices

This README document provides information on best practices that you should follow when you develop your applications to avoid introducing any security breaches. The document is divided into sections. Each section describes a use case scenario and possible vulnerabilities associated with it along with information on how to make sure that these vulnerabilities do not take place in you applications. Thus, you can use this document as check list to verify your applications security.

The example solution illustrates the described vulnerabilities and provides code samples with comprehensive. To launch this solution, you need to have DevExpress ASP.NET controls installed. You can download the installer from the DevExpress site.

## 1. Uploading and Displaying Binary Images

If there is a potential for a file containing a malicious script disguised as an image to be uploaded and sent to an end-user, an opportunity is open to run malicious code in the end-user’s browser (XSS via Content-sniffing)

In the example solution, you can reproduce the security issue using the following steps:

1. Run the solution and open the **UploadingBinaryImage/UploadControl.aspx** page.
2. Upload the **\App_Data\TestData\Content-Sniffing-XSS.jpg** file, which is a JavaScript file emulating a malicious script disguised as a JPEG image.
3. Open the **UploadingBinaryImage/UploadControl.aspx** page, which writes the uploaded file to the server response in the code behind.
4. As the result, java script code from the uploaded file is executed by the browser:

To mitigate the vulnerability, consider one of the following solutions:

1.	Programmatically check whether the uploaded file is really an image before saving it to the server-side storage.
2.	Use the [ASPxBinaryImage](https://documentation.devexpress.com/AspNet/11624/ASP-NET-WebForms-Controls/Data-Editors/Editor-Types/ASPxBinaryImage/Overview/ASPxBinaryImage-Overview) control for image uploading. This control implements automatic file type check.

It is also recommended that you always specify the exact content type when you write binary data to the response:

**Correct:** `Response.ContentType = "image"`;

**Potential security breach:** `Response.ContentType = "image/jpeg"`.

### Notes
1. Microsoft Edge automatically detects file type based on its content, which prevents execution of malicious scripts.
2. Make sure to strictly specify the maximum uploaded file size to prevent DoS attacks based on uploading large files.



## 2. Uploading Files
This section provides information on possible vulnerabilities that may occur when using controls with file uploading functionality ([ASPxBinaryImage](https://documentation.devexpress.com/AspNet/11624/ASP-NET-WebForms-Controls/Data-Editors/Editor-Types/ASPxBinaryImage/Overview/ASPxBinaryImage-Overview), [ASPxUploadControl](https://documentation.devexpress.com/AspNet/4040/ASP-NET-WebForms-Controls/File-Management/File-Upload/Overview/ASPxUploadControl-Overview), [ASPxFileManager](https://documentation.devexpress.com/AspNet/9030/ASP-NET-WebForms-Controls/File-Management/File-Manager/Overview/ASPxFileManager-Overview), [ASPxHtmlEditor](https://documentation.devexpress.com/AspNet/4024/ASP-NET-WebForms-Controls/HTML-Editor), [ASPxRichEdit](https://documentation.devexpress.com/AspNet/17721/ASP-NET-WebForms-Controls/Rich-Text-Editor), [ASPxSpreadsheet](https://documentation.devexpress.com/AspNet/16157/ASP-NET-WebForms-Controls/Spreadsheet)).

### 2.1. Uploading executables
See the **UploadingFiles/UploadControl.aspx** page source code for a full code sample with commentaries.

The vulnerability arises when a web application allows uploading executable files, which than can be executed on the server side. For example, a malefactor can upload a Web Form file to a directory accessible through and existing route and predict its URL. Sending a http request to this file will then cause its code to be invoked on the server side.

In this example solution, you can reproduce the security issue using the following steps:

1. Run the solution and open the **UploadingFiles/UploadControl.aspx** page.
2. Upload the **\App_Data\TestData\Malicious.aspx**, file.
3. Now it is possible to execute the uploaded file on the server side by reqesting it using the following address: **/UploadingFiles/Images/Malicious.aspx**

Take into account the following rules to mitigate this vulnerability:

1. Perform server-side validation of the uploaded file type by specifying the ASPxUploadControl.AllowedExtensions property.
2. Disable file execution in the upload folder ([https://stackoverflow.com/questions/3776847/how-to-restrict-folder-access-in-asp-net])

### 2.2. Getting unauthorized access to an uploaded file
See the **UploadingFiles/UploadControlTempFileName.aspx** page source code for a full code sample with commentaries. The vulnerability exists when a malefactor can potentially guess the path of an uploaded static file.

Take into account the following rules to mitigate this vulnerability when storing temporary files, which should not be accessed by third parties:

1. Use a dedicated file extension for temporary files on the server (for example “.tmp”). 
2. Consider assigning random file names using the [GetRandomName](https://msdn.microsoft.com/en-us/library/system.io.path.getrandomfilename(v=vs.110).aspx) method. Not that in the **Advanced** uploading mode, files are loaded in small pieces (200KB by default), thus setting the **httpRuntime**>**maxRequestLength** and **requestLimits**>**maxAllowedContentLength** options in **web.config** is not enough to prevent attacks.

### 2.3. Uploading big files – possible memory overflow and disk space cluttering
See the **UploadingFiles/UploadControlMemory.aspx** page for source code a full code sample with commentaries.

If the application does not restrict the maximum uploaded file size, there is a security breach allowing a malefactor to perform a denial of service (DoS) attack by cluttering up server memory and disk space.

Take into account the following rules to mitigate this vulnerability:

1. When working with massive uploaded files, access file contents using the [FileContent](http://help.devexpress.com/#AspNet/DevExpressWebUploadedFile_FileContenttopic) property (a Stream) rather than [FileBytes](http://help.devexpress.com/#AspNet/DevExpressWebUploadedFile_FileBytestopic) property (a byte array).
2. Specify the maximum size for uploaded files using the [UploadControlValidationSettings.MaxFileSize](http://help.devexpress.com/#AspNet/DevExpressWebUploadControlValidationSettings_MaxFileSizetopic) property.




























