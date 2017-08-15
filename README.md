# azabu
イケてるオブジェクトマッピングのためのDSLとその処理系

## examples

### Configure

```
task
+-task.conf
+-datatypes
| \-ListTsv
\-converters
  \-getAmazonUrl.conf
```

task.conf
``` 
name : "Santa's Present List Tokyo" // Descriptive name

reciever : {
  type : REST
  url : http://localhost:8080/requestForm
  method : GET
  key : query
}

input : {
	type : MyListTsv // Datatypes are modifiable -> datatypes/${type}.conf
	format : TSV // JSON, XML, ... etc
}

output : {
	type : KeyValuePair // Predefined types are also available
	format : TSV // JSON, XML, ... etc
}

sender : {
   type : redis
   config : ${global.redis-conf} 
}

filter : [
	range : {
		target : age  // name of column (TSV), name of key (JSON), or etc. 
		min-include : 0
		max-exclude : 20
	}
	match : {
		target : address // name of column (TSV), name of key (JSON), or etc. 
		pattern : ".*(T|t)okyo$" // regex pattern
	}
	compare : {
		left : good_deed_score // name of column (TSV), name of key (JSON), or etc. 
		right : bad_deed_score // name of column (TSV), name of key (JSON), or etc. 
		relation : grater_than_or_equals
	}
]

convert : {
	refs : getAmazonUrl // converters/${refs}.conf
}
``` 

datatypes/MyListTsv.conf
```
type:TSV
column:[name, address, age, desiredPresentsASIN, good_deed_score, bad_deed_score]
```

converters/getAmazonUrl.conf
```
key : input:address
value : "https://www.amazon.co.jp/dp/" + input:desiredPresentsASIN + "/"
```

### Build

```sh
$ tar -zcvf target
$ java -jar azabu.jar target
Server is running.
```

### Run

```sh
$ wget http://localhost:8080/requestForm?q=John%09Azabu,%20Minato,%20Tokyo%0915%09XXX111XXX111%09100%090
$ wget http://localhost:8080/requestForm?q=Taro%09Kyoto%0915%09YYY222YYY222%09100%090
$ Redis
> GET "Azabu, Minato, Tokyo"
https://www.amazon.com/dp/XXX111XXX111
> GET "Kyoto"
(nil)
```

