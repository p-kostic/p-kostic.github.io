---
title: Online tool to read base64 encoded gzip strings to decompress EF Migration History EDMX  
date: 2020-11-6 19:22:30 +0100
categories: [dotnet]
tags: [entity framework, csharp, dotnet, ms sql]     # TAG names should always be lowercase
---

## Motivation

While working with EF migrations, I found it quite irritating to view the EDMX that Entity framework produces in the Migration History table. It is not really an issue when you're already connected to that database context, as some C# code can be written to decompress it and return it as an `XDocument`. For example,

```csharp
private static XDocument Decompress(byte[] bytes)
{
    using (var memoryStream = new MemoryStream(bytes))
    {
        using (var gzipStream = new GZipStream(memoryStream, CompressionMode.Decompress))
        {
            return XDocument.Load(gzipStream);
        }
    }
}
```
[Source](https://stackoverflow.com/a/16549735/4516065)

However, depending on the type of project your working on, this requires you to create a separate endpoint, IO process, or start a debug process to inspect the decompressed XML in a readable format. 

When you're not connected to the database, all you have is the Azure Query editor or SQL Server Management Studio (SSMS). There probably is some SQL command that decompresses it, but the result isn't properly printed, as it would return a table. You'll also have to copy and paste it over to a text editor and have an XML formatter installed to format it automatically, because you don't want to do that manually. 

Finally, you still have the option to write a small script yourself that takes the copied value from either SSMS or the Azure Query editor and decompress it. To make this all a bit easier, I've created a way to do this through some Javascript within this blog post.

## How it's done
The EDMX is typically stored as a base64 string, which contains gzipped XML data. However, this depends on which program you're using to view the model field in the `dbo.__MigrationHistory` table.

* For the Azure Query Editor, you get the base64 string as a result (e.g. `H4sIA...`). Note that you have to export the query results to get the full length string copied to your OS's clipboard.
* For SSMS, you get the raw hex string with the 0x delimiter (e.g. `0x1F8B...`). Note that you have to right click and copy the cell to get the full length string copied to your OS's clipboard


The small JavaScript code in this Markdown file first determines what type of string it's dealing with. It will first check for the `0x` delimiter. In all other cases it will assume a base64 encoding


* In the event of base64, `window.atob()` is used to decode the base64 encoded string first. After converting the characters to ASCII codes in an `Uint8Array`,
* In the event of raw hex string, I use JavaScript's `parseInt()` function to go from hex to decimal in an `Uint8Array`.


 I finally use [pako.js](https://github.com/nodeca/pako) to decompress the gzip `Uint8Array`

 
 The source code can be found [here](https://raw.githubusercontent.com/p-kostic/p-kostic.github.io/master/_posts/dotnet/2020-11-6-tool-to-read-gzip-strings-to-decompress-entity-framework-migration-history-edmx.md)

<script src="../../assets/js/pako.min.js">

</script>

## Try it out!

```js
// example base64, copy the line below and paste it in the input field
H4sIAAAAAAAAAwXB2w0AEBAEwFbWl2Y0IW4jQmziPNo3k6TuGK0Tj/ESVRs6yzkuHRnGIqPB92qzhg8yp62UMAAAAA==
```

> Input: Paste your model string here and hit the 'Decompress' button

<textarea id="modelInput" name="modelInput" rows="10" cols="120" style="overflow: auto; max-width: 100%; -webkit-box-sizing: border-box; -moz-box-sizing: border-box; box-sizing: border-box;"></textarea>

<button id="decompressButton">Decompress</button>

> Output (Note that something is wrong with your model string if it returns undefined)

<textarea id="modelOutput" name="modelOutput" rows="30" cols="120" style="overflow: auto; max-width: 100%; -webkit-box-sizing: border-box; -moz-box-sizing: border-box; box-sizing: border-box;" wrap='off'></textarea>

## Final remarks
Unfortunately, I'm unable to do any XML syntax highlighting on the output because this all made written in a Markdown file. You'll have to do that by copying it to your favorite editor. If you do know how to do syntax highlighting after the fact, with text coming from an EventListener in JavaScript, let me know!

<script>

var fromHexString = function(hexString) {
    hexString = hexString.replace(/^0x/, '');
    return new Uint8Array(hexString.match(/.{1,2}/g).map(byte => parseInt(byte, 16)));
};

var handleBase64 = function(inputString) {
    try {
        var bytes = [];

        for (var i = 0; i < inputString.length; i++) {
            var abyte = inputString.charCodeAt(i) & 0xff;
            bytes.push(abyte);
        }

        var binData = new Uint8Array(bytes);
        var data = pako.inflate(binData, { to: 'string' });

    } catch (e) {
        document.getElementById("modelOutput").value = e;
        console.error(e);
    }
    return data;
};

var handleHex = function(inputString) {
    try {
        var data = pako.inflate(inputString);
        return String.fromCharCode.apply(null, new Uint8Array(data));
    } catch (e) {
        document.getElementById("modelOutput").value = e;
        console.error(e);
    }
};

document.getElementById("decompressButton").addEventListener("click", function() {

    var inputString = document.getElementById("modelInput").value;

    if (inputString === "") {
        document.getElementById("modelOutput").value = "No input was given";
        return;
    }

    if (inputString.startsWith("0x")) {
        inputString = fromHexString(inputString);
        document.getElementById("modelOutput").value = handleHex(inputString);
    } else {
        inputString = atob(inputString);
        document.getElementById("modelOutput").value = handleBase64(inputString);
    }
});

</script>