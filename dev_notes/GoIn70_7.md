
# 7 Serving and receiving assets and forms

## 7.1 Serving static content

 - To handle static files, the http package in the standard library has a series of functions that deal with file serving. 

```
// Listing 7.1 http package file serving: file_serving.go

package main
import (
     "net/http"
)
func main() {
    // Uses a directory on the filesystem
    dir := http.Dir("./files")
    // Serves the filesystem directory
    http.ListenAndServe(":8080", http.FileServer(dir))
}
```

 - From a directory on the local filesystem, FileServer will serve files following proper permissions.
 - It’s capable of looking at the `If-Modified-Since` HTTP header and responding with a 304 Not Modified response if the version of the file a user already has matches the one currently being served.

 - When you want to write your own handler to serve files, the ServeFile function in the http package is useful, as shown in the next listing.

```
// Listing 7.2 Serve file with custom handler: servefile.go

// Registers a handler for all paths
func main() {
     http.HandleFunc("/", readme)
     http.ListenAndServe(":8080", nil)
}

// Serves the contents of a readme file
func readme(res http.ResponseWriter, req *http.Request) {
     http.ServeFile(res, req, "./files/readme.txt")
}
```

 - This readme handler serves the content of a file located at ./files/readme.txt by using the ServeFile function. 
 - And like FileServer, ServeFile looks at the If-Modified-Since HTTP header and responds with a 304 Not Modified response if possible.
 - This functionality, along with some of its underpinnings, enables you to serve con- tent by using a variety of techniques.


####  TECHNIQUE 39 Serving subdirectories

```
// Listing 7.3 Serving a subdirectory
func main() {
    // A directory and its subdirectories 
    // on the filesystem are chosen to serve.
    dir := http.Dir("./files/")
    // The /static/ path serves the directory and 
    // needs to be removed before looking up file path.
    handler := http.StripPrefix("/static/", http.FileServer(dir))
    http.Handle("/static/", handler)

    // Serves a home page that may include
    // files from the static directory
    http.HandleFunc("/", homePage)
    http.ListenAndServe(":8080", nil)
```

 - Here, the built-in web server is serving the ./files/ directory at the path /static/ by using the file server from the http package. 


## 7.2 Handling form posts

### 7.2.1 Introduction to form requests

 - When a request is made to a server and it contains form data, that request isn’t processed into a usable structure by default. 
 - The following example shows the simplest way to parse form data and get access to it:

```
func exampleHandler(w http.ResponseWriter, r *http.Request) {
     name := r.FormValue("name")
}
```

 - Behind this call to `FormValue`, a lot is going on.
    - FormValue starts by parsing the form data into a Go data structure. It’s looking to parse text form data and multipart form data, such as files.
    - After the data is parsed, it looks up the key (form field name) and returns the first value for the key, if one exists.
    - If there’s nothing with this key, an empty string is returned.
 - Although this case makes it look easy, a lot is going on that you may not want, and there are features that you can’t access here.
    - For example, what if you want to skip looking for multipart form data and trying to parse it because you know it won’t be present?
    - Or what if a form field has multiple values, and you want to get at all of them?
 - The first step to work with form data is to parse it .
    - Inside a request handler are two methods on the Request object that can parse form data into a Go data structure. 
        - The ParseForm method parses fields that contain text. 
        - If you need to work with binary data or files from the form, you need to use ParseMultipartForm.
            - As its name suggests, this method works on multipart form data (a form containing con- tent with different MIME content types).
            - ParseMultipartForm is called by FormValue in the preceding example if parsing hasn’t happened yet.
 - The form data is parsed into two locations:
    - The Form property on the Request object will contain the values from the URL query along with the values submitted as a POST or PUT body. 
        - Each key on Form is an array of values. 
        - The FormValue method on Request can be used to get the first value for a key. 
    - When you want the values from the POST or PUT body without those from the URL query, you can use the `PostForm` property on the Request object. 
        - Like FormValue, the `PostFormValue` method can retrieve the first value from PostForm for a key.


```
// Listing 7.10 Parsing a simple form response

func exampleHandler(w http.ResponseWriter, r *http.Request) {
    err := r.ParseForm()
    if err != nil {
        fmt.Println(err)
    }
    // Gets the first value for the name field from the form
    name := r.FormValue("name")
}
```

 - This listing contains the handling for a simple form. 
    - This simple example works for forms with only text fields. 
    - If a file field were present, it wouldn’t be parsed or accessible.
    - And it works only for form values that have a single response. HTML forms allow for multiple responding values. 

#### TECHNIQUE 44 Accessing multiple values for a form field

 - PROBLEM: FormValue and PostFormValue each return the first value for a form field. 
    - When you have multiple values, how can you access all of them?
 - SOLUTION: Instead of using FormValue and PostFormValue to retrieve a field value, look up the field on the `Form or PostForm properties` on the Request object. Then iterate over all the values.

```
// Listing 7.11 Parsing a form with multiple values for a field

func exampleHandler(w http.ResponseWriter, r *http.Request) {
    // The maximum memory to store file parts, where rest is stored to disk
    maxMemory := 16 << 20
    err := r.ParseMultipartForm(maxMemory)
    if err != nil {
        fmt.Println(err)
    }
    for k, v := range r.PostForm["names"] {
        fmt.Println(v)
    }
}
```

 - The default number used when FormValue or PostFormValue needs to call ParseMultipartForm is 32 megabytes.
 - The names field on the PostForm property is used, limiting the values to just those submitted in the POST or PUT body.

### 7.2.2 Working with files and multipart submissions

#### TECHNIQUE 45 Uploading a single file

 - PROBLEM: When a file is uploaded with a form, how to you process and save it?
 - SOLUTION: When a file is uploaded, process the form as a multipart form by using `ProcessMultipartForm` on the Request object.
    - This picks up the file parts. 
    - Then use the FormFile method on the Request object to access and file fields, uploading a single file.
    - For each file, you can access the metadata and a file object that’s similar to File objects from the os package.
 - Handling a file is nearly as straightforward as handling text form data.
    - The difference lies in the binary file and the metadata surrounding it, such as the filename.
    - The following listing presents a simple file-upload form.


```
// Listing 7.12 A form with a single-value file-upload field

<!doctype html>
<html>
  <head>
    <title>File Upload</title>
  </head>
  <body>
      <form action="/" method="POST" enctype="multipart/form-data">
        <label for="file">File:</label>
        <input type="file" name="file" id="file">
        <br>
        <button type="submit" name="submit">Submit</button>
      </form>
  </body>
</html>
```

 - This form has some important parts. The form method is POST, and its encoding is in multipart.
    - Being multipart allows the text part of the form to be uploaded and processed as text, while the file is handled using its own file type.
    - The input field is typed for a file, which tells browsers to use a file picker and upload the contents of the file. 
 - This form is served and processed by the handler function for the http package in the following listing.


```
// Listing 7.13 Handle a single file upload

// http handler to display and process the form in file.html
func fileForm(w http.ResponseWriter, r *http.Request) {
    // When the path is accessed with a GET
    // request, displays the HTML page and form
    if r.Method == "GET" {
        t, _ := template.ParseFiles("file.html")
        t.Execute(w, nil)
    } else {
        // Gets the file handler, header information, and
        // error for the form field keyed by its name
        f, h, err := r.FormFile("file")
        if err != nil {
            panic(err)
        }
        defer f.Close()
        filename := "/tmp/" + h.Filename
        out, err := os.Create(filename)
        if err != nil {
            panic(err)
        }
        defer out.Close()
        // Copies the uploaded file to 
        // the local location
        io.Copy(out, f)
        fmt.Fprint(w, "Upload complete")
    } 
}
```

 - The first step used to process the file field is to retrieve it by using the FormFile method on the Request.
    - If the form hasn’t been parsed, FormFile will call Parse- MultipartForm. 
    - FormFile then returns a `multipart.File` object, a `*multipart .FileHeader` object, and an error if there is one.
    - The `*multipart.FileHeader` object has a Filename property that it uses here as part of the location on the local filesystem to store the upload.
 - This solution works well for a field with a single file.
    - HTML forms allow for multi- value fields, and this solution will pick up only the first of the files. 
    - For multivalue file uploads, see the next technique.


#### TECHNIQUE 46 Uploading multiple files

 - PROBLEM: How do you process the files when multiple files are uploaded to a single file-input field on a form?
 - SOLUTION: Instead of using FormFile,   parse the form and retrieve a slice with the files from the `MultipartForm` property on the Request.  Then iterate over the slice, individually handling each file.

```
// Listing 7.14 A form with a multiple value file-upload field

<!doctype html>
<html>
  <head>
    <title>File Upload</title>
  </head>
  <body>
    <form action="/" method="POST" enctype="multipart/form-data">
      <label for="files">File:</label>
      <input type="file" name="files" id="files" multiple>
      <br>
      <button type="submit" name="submit">Submit</button>
    </form>
</body>
</html>
```

 - The `multiple` attribute turns a single file-input field into one accepting multiple files. 

```go
// Listing 7.15 Process file form field with multiple files

func fileForm(w http.ResponseWriter, r *http.Request) {
    if r.Method == "GET" {
        t, _ := template.ParseFiles("file_multiple.html")
        t.Execute(w, nil)
    } else {
        err := r.ParseMultipartForm(16 << 20)
        if err != nil {
            fmt.Fprint(w, err)
            return
        }
        data := r.MultipartForm
        files := data.File["files"]
        for _, fh := range files {
            f, err := fh.Open()
            defer f.Close()
            if err != nil {
                fmt.Fprint(w, err)
                return
            }
            out, err := os.Create("/tmp/" + fh.Filename)
            defer out.Close()
            if err != nil {
                fmt.Fprint(w, err)
                return
            }
            _, err = io.Copy(out, f)
            if err != nil {
                fmt.Fprintln(w, err)
                return
            }
        }
        fmt.Fprint(w, "Upload complete")
    }
}
```

#### TECHNIQUE 47 Verify uploaded file is allowed type

 - When a file is uploaded, it could be any type of file. 
     - The upload field could be expecting an image, a document, or something else altogether.
     - But is that what was uploaded? How would you handle an improper file being uploaded?
 - PROBLEM: How can you detect the type of file uploaded to a file field inside your application?
 - SOLUTION: To get the MIME type for a file, you can use one of a few ways, with varying degrees of trust in the value:
    - When a file is uploaded, the request headers will have a Content-Type field with either a specific content type, such as image/png, or a general value of application/octet-stream.
    - A file extension is associated with a MIME type and can provide insight into the type of file being uploaded.
    - You can parse the file and detect the content type based on the contents of the file.
 - These first two methods rely on outside parties for accuracy and trust. 
    - The third solution requires parsing the file and knowing what to look for to map to a content type. This is the most difficult method and uses the most system resources, but is also the most trusted one


```
// he content type here will either be a specific MIME type, 
// such as image/png, or a generic value of 
// application/octet-stream when the type was unknown.
file, header, err := r.FormFile("file")
contentType := header.Header["Content-Type"][0]
```

```
file, header, err := r.FormFile("file")
extension := filepath.Ext(header.Filename)
type := mime.TypeByExtension(extension)
```
 
 - The http package contains the function DetectContentType, capable of detecting the type for a limited number of file types. 
    -  These include HTML, text, XML, PDF, PostScript, common image formats, com- pressed files such as RAR, Zip, and GZip, wave audio files, and WebM video files.

```
file, header, err := r.FormFile("file")
buffer := make([]byte, 512)
_, err = file.Read(buffer)
filetype := http.DetectContentType(buffer)
```

 - The buffer is only 512 bytes because DetectContentType looks at only up to the first 512 bytes when determining the type. When it isn’t able to detect a specific type, application/octet-stream is returned.


### 7.2.3 Working with raw multipart data

 - The previous file-handling techniques work well when you’re dealing with small files or files as a whole, but limit your ability to work with files while they’re being uploaded.
    - For example, if you’re writing a proxy and want to immediately transfer the file to another location, the previous techniques will cache large files on the proxy.
 - The Go standard library provides both high-level helper functions for common file-handling situations, and lower-level access that can be used for the less common ones or when you want to define your own handling.
 - Instead of using the ParseMultipartForm method on the Request object inside an http handler function, you can access the raw stream of the request by accessing the underlying `*multipart.Reader` object. 
    - This object is accessible by using the MultipartReader method on the Request.
 

#### TECHNIQUE 48 Incrementally saving a file

 - 
