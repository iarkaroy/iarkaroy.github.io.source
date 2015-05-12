---
title: "Download ZIP File Dynamically with PHP"
layout: post
excerpt: We will see how we can make a webpage act as an initializer to download a zip file. We will just provide the location of the file and PHP will download it to the user. In the back-end, the HTTP headers are responsible for the download. We will set the headers with PHP.
---
ZIP is a very popular archive file format that supports lossless data compression. A .ZIP file may contain one or more files or folders that may have been compressed. ZIP files are used in almost every downloadable content as it is widely supported by almost all operating systems.

ZIP files generally use the file extensions ".zip" or ".ZIP" and the MIME media type `application/zip`. ZIP is used as a base file format by many applications and tools. Most popular among them is WordPress. They use zip file format for themes, plugins and the application itself. And everything is done dynamically.

Here we will see how we can make a webpage act as an initializer to download a zip file. We will just provide the location of the file and PHP will download it to the user. In the back-end, the HTTP headers are responsible for the download. We will set the headers with PHP.

    $filename = "My Awesome Zip File.zip";
    $filepath = "/var/www/downloads/file.zip";

The `$filename` contains the name that will appear to the user and `$filepath` contains the full absolute path where the file is physically located.

Now we will set the headers with PHP [header()](http://php.net/manual/en/function.header.php). And whenever any request comes to this page, browser will read the header response and start the download immediately.

    header("Pragma: public");
    header("Expires: 0");
    header("Cache-Control: must-revalidate");
    header("Cache-Control: public");
    header("Content-Description: File Transfer");
    header("Content-type: application/octet-stream");
    header('Content-Disposition: attachment; filename="' . $filename . '"');
    header("Content-Transfer-Encoding: binary");
    header("Content-Length: " . filesize($filepath));
    ob_end_flush();
    @readfile($filepath);

Of course, the name of the downloaded file will be *My Awesome Zip File.zip*.