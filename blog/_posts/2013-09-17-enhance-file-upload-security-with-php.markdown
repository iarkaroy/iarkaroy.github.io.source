---
title: "Enhance File Upload Security with PHP"
layout: post
tags:
- php
- security
- upload
---
We often face a situation where we need to provide our users a way to upload image, pdf, document etc. The basic html form for file upload is able to handle this with the help of PHP. The basic process of uploading user submitted files to the server with PHP is fairly easy and simple. But there should be some security measures. What will happen if a bad user uploads a malicious file to our server?

Here we will discuss some of this security measures to block the holes.

## The Form

Before anything else, lets assume we have a form for the users to upload a file.

    <form action="<?php echo $_SERVER['PHP_SELF']; ?>" method="post" enctype="multipart/form-data">
        <label for="file">Select File:</label>
        <input type="file" name="file" id="file">
        <input type="submit" name="upload" value="Upload File">
    </form>

Remember to use `method="post"` whenever using a form for file upload. As you are already aware of, `enctype="multipart/form-data"` is necessary because it tells the browser how to handle the data submitted as a file.

<br>

## `$_FILES` Array

As soon as the file uploads, its details become available in the PHP's `$_FILES` superglobal array.

If you add the following lines at the top of your page,

    <?php
        if(isset($_POST['upload'])) { // Checking whether the form has been submitted
            echo '<pre>';   
            print_r($_FILES);
            echo '</pre>';
        }
    ?>

and then you upload a file named flower.jpg. After you click 'Upload File' button, the details would be shown like this:

    Array
    (
        [file] => Array
            (
                [name] => flower.jpg
                [type] => image/jpeg
                [tmp_name] => C:\xampp\tmp\php9F19.tmp
                [error] => 0
                [size] => 780831
            )

    )

<br>

## The Errors

Before we continue, lets take a look at the errors we can have. In the previous example, the `error` is `0` which means that there is no error and the file has been successfully uploaded.

Lets take a look at the various error codes and what they mean.

    0 => No Error
    1 => Exceeds upload_max_filesize in PHP configuration
    2 => Exceeds MAX_FILE_SIZE in hidden form field
    3 => File partially uploaded
    4 => No file to upload
    6 => No temporary upload folder defined
    7 => PHP failed to write to disk
    8 => A PHP extension prevent the file to be uploaded

<br>

## MAX_FILE_SIZE

In some cases, you might want to limit the size of the file to be uploaded. The `MAX_FILE_SIZE` field tells the browser to prevent the upload if it exceeds certain size. To use this feature, add the following code before the file input field.

    <?php
        $max_size = 100 * 1024; // 100KB
    ?>
    <input type="hidden" name="MAX_FILE_SIZE" value="<?php echo $max_size; ?>">

If a file larger than the size set, the error will be set to `2` and it will not be attempted to upload.

<br>

## Upload Directory

When setting or specifying a directory to store the uploaded files, it is much professional to consider some points before doing so.

The directory where the uploaded file will live, must be set writable. In Linux, permission to the folder must be set to at least 755 or 775.

If the files are going to be accessible by public, the directory can be under the root of the server. But if the files are protected or contains sensitive data, then it is much wiser to put the directory outside the scope of the server root so that it cannot be accessible via public url.

<br>

## Move Uploaded File

It is necessary to move the uploaded file to the directory specified for the files. Otherwise, the uploaded file will be lost as it is just saved as a temporary file at first.

To move the uploaded file, we will use `move_uploaded_file()` method:

    <?php
        if(isset($_POST['upload'])) { // Checking whether the form has been submitted
            ...
            $directory = __DIR__ . '/uploads/';
            if($_FILES['file']['error'] == 0) { // No error
                $result = move_uploaded_file($_FILES['file']['tmp_name'], $directory . $_FILES['file']['name']);
                if($result) {
                    echo $_FILES['file']['name'] . ' has been successfully uploaded.';
                } else {
                    echo 'There was a problem uploading ' . $_FILES['file']['name'];
                }
            }
            ...
        }
    ?>

Now the uploaded files will the saved in the 'uploads' directory.

We have improved to some extent but there is a long path to go. We have not implemented any check on file types, file names and we have not provided any support for uploading multiple files. Also, if the uploaded files have same filename, the older one will be overwritten.

To avoid these situations and implement more validation, we will handle the upload a bit differently. We will [Create a File Upload Class in PHP]({{site.url}}{% post_url 2013-09-29-create-file-upload-class-in-php %}) to handle the file upload more efficiently.