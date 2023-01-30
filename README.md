# Danis³h Short-Form Notation of XML and HTML
In addition to the standard Danis³h Notation of HTML and XML, Danis³h has a more concise short-form notation.

The short-form notation is not identical to, but inspired by the output produced by the combination of *two* functions native to **PHP**:

 - `simplexml_load_string()`
 - `json_encode()`

**e.g.**

```php
$My_XML_String = '<my-extensible><markup-language>String</markup-language></my-extensible>';
$My_XML_Object = simplexml_load_string($My_XML_String);
$My_XML_JSON = json_encode($My_XML_Object);
```

_____

The differences between the output of the PHP script immediately above and **Short-Form Danis³h Notation** are minor but important:

### Example XML
```xml
<note>
<to>Alice</to>
<from>Bob</from>
<heading>Question</heading>
<body>Have you seen my chocolate pretzels?</body>
</note>
```

### PHP Native Functions Output:
```json
{"to":"Alice","from":"Bob","heading":"Question","body":"Have you seen my chocolate pretzels?"}
```

### Danis³h Short-Form Notation
```json
{"note":{"to":{"text":"Alice"},"from":{"text":"Bob"},"heading":{"text":"Question"},"body":{"text":"Have you seen my chocolate pretzels?"}}}
```
    
_______

```php

function renameKeys($associativeArray) {

  $associativeArrayKeys = array_keys($associativeArray);

  for ($i = 0; $i < count($associativeArrayKeys); $i++) {

    $key = $associativeArrayKeys[$i];

    if (is_array($associativeArray[$key])) {

      $associativeArray[$key] = renameKeys($associativeArray[$key]);
    }

    $Previous_Refs = 0;
    $Key_Root = explode(':', $key)[0];

    for ($j = 0; $j < count($associativeArrayKeys); $j++) {

      if ($associativeArrayKeys[$j] === $key) break;
      $Previous_Refs = (explode(':', $associativeArrayKeys[$j])[0] === $Key_Root) ? ($Previous_Refs + 1) : $Previous_Refs;
    }

    $associativeArrayKeys[$i] = ($Previous_Refs < 1) ? explode(':', $associativeArrayKeys[$i])[0] : explode(':', $associativeArrayKeys[$i])[0].':'.($Previous_Refs + 1);
    $associativeArray[$associativeArrayKeys[$i]] = $associativeArray[$key];
    unset($associativeArray[$key]);
  }

  return $associativeArray;
}

function Build_Short_Form_Danis3h_Markup($shortformSource) {

  // ESCAPE SPACES IN TEXT
  $shortformSourceArray = explode('"plainText":', $shortformSource);

  for ($i = 1; $i < count($shortformSourceArray); $i++) {

    $subarray = explode('"', $shortformSourceArray[$i]);
    $subarray[1] = str_replace(' ', '␠', $subarray[1]);
    $shortformSourceArray[$i] = implode('"', $subarray);
  }

  $shortformString = implode('"plainText":', $shortformSourceArray);

  // GENERAL PREPARATION
  $shortformString = preg_replace('/\s/', '', $shortformString);
  $shortformString = preg_replace('/\"element\"\:(\"[^\"]*\")\,/', '$1:', $shortformString);
  $shortformString = str_replace('"plainText"', '"text"', $shortformString);
  $shortformString = str_replace(['[', '"elementChildren":', ']'], '', $shortformString);
  $shortformString = str_replace('"self-closing":true},{', '[],', $shortformString);
  $shortformString = str_replace('}},{', '},', $shortformString);
  $shortformString = preg_replace('/\:([^\{\[\"])/', ':[]$1', $shortformString);


  // DE-DUPLICATE INDEXES  
  $shortformArray = explode('":', $shortformString);

  for ($i = 0; $i < (count($shortformArray) - 1); $i++) {

    $shortformArray[$i] .= ':'.substr(str_shuffle(str_repeat('abcdefghijklmnopqrstuvwxyz', 3)), 0, 3).'":';
  }

  $shortformString = implode($shortformArray);

  // RENAME KEYS AND FINAL PREPARATION
  $shortformAssociativeArray = json_decode($shortformString, TRUE);
  $shortformAssociativeArray = renameKeys($shortformAssociativeArray);
  $shortformJSON = json_encode($shortformAssociativeArray, JSON_UNESCAPED_SLASHES | JSON_UNESCAPED_UNICODE);
  $shortformJSON = str_replace('␠', ' ', $shortformJSON);

  return $shortformJSON; 
}

Build_Short_Form_Danis3h_Markup($Danis3h_Long_Form_Markup);

```

_________

## Example HTML
```html
<section><div><p>This is a test.</p><p>This is another test.</p><nav></nav></div><div><p>This is a third test.</p><p>This is a fourth test.</p><hr/><span>Something else.</span><hr/><p>This is a fifth test.</p></div></section>
```


## Example Input (Long-form HTML Notation in Danis³h)
```json
[{"element":"section","elementChildren":[{"element":"div","elementChildren":[{"element":"p","elementChildren":[{"plainText":"This is a test."}]},{"element":"p","elementChildren":[{"plainText":"This is another test."}]},{"element":"nav","elementChildren":[]}]},{"element":"div","elementChildren":[{"element":"p","elementChildren":[{"plainText":"This is a third test."}]},{"element":"p","elementChildren":[{"plainText":"This is a fourth test."}]},{"element":"hr","self-closing":true},{"element":"span","elementChildren":[{"plainText":"Something else."}]},{"element":"hr","self-closing":true},{"element":"p","elementChildren":[{"plainText":"This is a fifth test."}]}]}]}]
```

## Example Output (Short-form HTML Notation in Danis³h)
```json
{"section":{"div":{"p":{"text":"This is a test."},"p:2":{"text":"This is another test."},"nav":[]},"div:2":{"p":{"text":"This is a third test."},"p:2":{"text":"This is a fourth test."},"hr":[],"span":{"text":"Something else."},"hr:2":[],"p:3":{"text":"This is a fifth test."}}}}
```
