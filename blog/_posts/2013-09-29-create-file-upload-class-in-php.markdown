---
title: "Create a File Upload Class in PHP"
layout: post
tags:
- php
- security
- upload
- class
---
The basic process of providing a HTML form for uploading user submitted files to the server with PHP is fairly easy and simple. But there are some security implications that many of us are unaware of. We will be building a custom PHP class for secure file upload. This class will check the type and size of the file and rename the file in case of duplication.

Those who are absolutely beginner in handling file upload, are requested to take a look at [Enhance File Upload Security with PHP]({{site.url}}{% post_url 2013-09-17-enhance-file-upload-security-with-php %}). That article will help you to better understand file uploading from scratch.

## The Class

We will start our `Upload` class with a blank constructor

    class Upload {
        function __construct() {

        }
    }

Now we will add some properties to our class to store some data.

    protected $config = array();
    protected $current;
    protected $errors = array();
    protected $new_name;

The `$config` property holds the configurations like target directory, size limit etc. `$current` refers to the current key of `$_FILES` array. `$errors` contains the errors encountered during process. `$new_name` stores the name of the uploaded file to be set.

## Public Methods

Now we are going to define some public methods to set up some configuration settings.

But first, we need to implement a method for handling errors.
    
    public function error($msg = null) {
        if ($msg) {
            $this->errors[] = $msg;
            return $this;
        } else {
            return $this->errors;
        }
    }

This method will store error messages in the `$errors`. This method will work as both a getter and setter.

### Extensions

Of course you want to prevent the users from uploading specified types of files. Here is the method for allowing or disallowing file types.

    public function setAllowedExtensions($extensions) {
        if (is_array($extensions)) {
            $this->config['allowed_extensions'] = $extensions;
        } else {
            $extensions = explode(',', $extensions);
            $this->config['allowed_extensions'] = array();
            foreach ($extensions as $ext) {
                $this->config['allowed_extensions'][] = trim($ext);
            }
        }
        return $this;
    }

    public function setDisallowedExtensions($extensions) {
        if (is_array($extensions)) {
            $this->config['disallowed_extensions'] = $extensions;
        } else {
            $extensions = explode(',', $extensions);
            $this->config['disallowed_extensions'] = array();
            foreach ($extensions as $ext) {
                $this->config['disallowed_extensions'][] = trim($ext);
            }
        }
        return $this;
    }

The allowed extensions will be stored in `$config['allowed_extensions']` and the disallowed extensions in `$config['disallowed_extensions']`. You can either specify allowed file types or disallowed file types.

### Target Directory

Now its time to specify where our uploaded images will live. We should add a method to setup this setting.

    public function setDirectory($dir) {
        $dir = str_replace('\\', '/', $dir);
        $dir = rtrim($dir, '/');
        $this->config['directory'] = $dir;
        return $this;
    }

### Max Size

Obviously we want to limit the size of the files to be uploaded. But our specified limit cannot be greater than that specified in PHP config.

    public function setMaxSize($size) {
        $max_size = $this->convertSizeToBytes($size);
        $upload_max_filesize = $this->convertSizeToBytes(ini_get('upload_max_filesize'));
        $this->config['max_size'] = ($max_size <= $upload_max_filesize) ? $max_size : $upload_max_filesize;
        return $this;
    }

This will check the size limit from PHP configuration. If the specified limit is larger, it will automatically set the size from PHP config. For this method to work properly, we need to implement a helper method.

    protected function convertSizeToBytes($size) {
        $size = trim($size);
        $unit = strtoupper($size[strlen($size) - 1]);
        if (in_array($unit, array('T', 'G', 'M', 'K', 'B'))) {
            switch ($unit) {
                case 'T':
                    $size *= 1024;
                case 'G':
                    $size *= 1024;
                case 'M':
                    $size *= 1024;
                case 'K':
                    $size *= 1024;
                case 'B':
                    $size *= 1;
            }
        }
        return $size;
    }

### Overwrite

We need provide a way to specify whether we want to overwrite files or not.

    public function setOverwrite($overwrite = TRUE) {
        $this->config['overwrite'] = $overwrite;
        return $this;
    }

## Methods for the Magic

The methods mentioned till now are going to be used to set the configuration settings. For the actual file uploading process, we need to add a few more internal methods to check the settings or generate a new name for the file or upload the file etc.

    // Check whether the file is of allowed type
    protected function allowedExtension() {
        $allowed = TRUE;
        if (!isset($this->config['allowed_extensions']) || !$this->config['allowed_extensions']) {
            $allowed = TRUE;
        }
        if (!isset($this->config['disallowed_extensions']) || !$this->config['disallowed_extensions']) {
            $allowed = TRUE;
        }
        $ext = pathinfo($this->current['name'], PATHINFO_EXTENSION);
        $ext = strtolower($ext);
        if (isset($this->config['allowed_extensions']) && $this->config['allowed_extensions']) {
            $allowed = in_array($ext, $this->config['allowed_extensions']);
        }
        if (isset($this->config['disallowed_extensions']) && $this->config['disallowed_extensions']) {
            $allowed = !in_array($ext, $this->config['disallowed_extensions']);
        }
        return $allowed;
    }

    // Check whether the file is within size limit
    protected function allowedSize() {
        $size = $this->current['size'];
        return $size <= $this->config['max_size'];
    }

    // Validate the target directory
    protected function checkDirectory() {
        if (!isset($this->config['directory']) || empty($this->config['directory'])) {
            return FALSE;
        }
        if (!is_dir($this->config['directory'])) {
            mkdir($this->config['directory']);
        }
        return is_writable($this->config['directory']);
    }

    // Generate new name for the file to be saved
    // it will replace spaces with dashes
    // name will be lowercase
    // generate new name if file exists
    protected function setNewName() {
        $filename = $this->current['name'];
        $name = pathinfo($filename, PATHINFO_FILENAME);
        $ext = pathinfo($filename, PATHINFO_EXTENSION);
        $name = str_replace(' ', '-', $name);
        $name = strtolower($name);
        $this->new_name = $name . '.' . $ext;
        if (isset($this->config['overwrite']) && $this->config['overwrite']) {
            return $this;
        }
        $count = 0;
        while (file_exists($this->config['directory'] . '/' . $this->new_name)) {
            $count++;
            $this->new_name = $name . '_' . $count . '.' . $ext;
        }
        return $this;
    }

    // Move uploaded file with new name
    protected function moveFile() {
        $this->setNewName();
        return move_uploaded_file($this->current['tmp_name'], $this->config['directory'] . '/' . $this->new_name);
    }

    // Sets error messages according to upload error
    protected function uploadError() {
        switch ($this->current['error']) {
            case 1:
            case 2:
                $this->error('The file is bigger than specified limit.');
                break;
            case 3:
                $this->error('The file is uploaded partially. Please try again.');
                break;
            case 4:
                $this->error('No file selected to upload.');
                break;
            case 6:
                $this->error('No upload directory has been specified.');
                break;
            default :
                $this->error('Failed to upload the file.');
                break;
        }
    }

    // Public method to execute the upload process
    public function run() {
        $this->current = current($_FILES);
        if ($this->current['error'] != 0) {
            $this->uploadError();
            return FALSE;
        }
        if (!$this->allowedExtension()) {
            $this->error('The type of the file is not supported.');
            return FALSE;
        }
        if (!$this->allowedSize()) {
            $this->error('The size of the file is bigger than limit.');
            return FALSE;
        }
        if (!$this->checkDirectory()) {
            $this->error('Unable to access upload directory');
            return FALSE;
        }
        if ($this->moveFile()) {
            return $this->new_name;
        }
        return FALSE;
    }

## Adding power to the Constructor

While we are able set configuration settings by calling respective methods, it will be more comprehensive if we can provide an option to set the configuration in the constructor.

    function __construct($config = array()) {
        if ($config && !is_array($config)) {
            return FALSE;
        }
        foreach ($config as $option => $value) {
            $parts = explode('_', $option);
            $cap = array_map('ucfirst', $parts);
            $method = 'set' . implode("", $cap);
            if (method_exists($this, $method)) {
                call_user_func_array(array($this, $method), (array) $value);
            }
        }
    }

## Usage

To use this class, lets have a form first.

    <form action="<?php echo $_SERVER['PHP_SELF']; ?>" method="post" enctype="multipart/form-data">
        <label for="file">Select File:</label>
        <input type="file" name="file" id="file">
        <input type="submit" name="upload" value="Upload File">
    </form>

Add following lines of code at the top of the document.

    if(isset($_POST['upload'])) {
        require 'upload.php';
        $upload = new Upload([
            'overwrite' => FALSE,
            'max_size' => '5M',
            'directory' => __DIR__ . '/uploads/',
            'disallowed_extensions' => 'exe, bin, sh, py',
        ]);
        $response = $upload->run();
        if(!$response) {
            $errors = $upload->error();
        }
    }