# node-mssql [![Dependency Status](https://david-dm.org/patriksimek/node-mssql.png)](https://david-dm.org/patriksimek/node-mssql) [![NPM version](https://badge.fury.io/js/mssql.png)](http://badge.fury.io/js/mssql)

[![NPM](https://nodei.co/npm/mssql.png)](https://nodei.co/npm/mssql/)

An easy-to-use MSSQL database connector for NodeJS.

There are some TDS modules which offer functionality to communicate with MSSQL databases but none of them does offer enough comfort - implementation takes a lot of lines of code. So I decided to create this module, that make work as easy as it could without loosing any important functionality.

At the moment it support two TDS modules:
* [Tedious](https://github.com/pekim/tedious) by Mike D Pilsbury (pure javascript - windows/osx/linux)
* [Microsoft Driver for Node.js for SQL Server](https://github.com/WindowsAzure/node-sqlserver) by Microsoft Corporation (native - windows only)

## Installation

    npm install mssql

## Quick Example

```javascript
var sql = require('mssql'); 

config = {
	user: '...',
	password: '...',
	server: 'localhost',
	database: '...'
}

var connection = new sql.Connection(config, function(err) {
	
	// Query
	
	var request = new sql.Request(connection); // or: var request = connection.request();
	request.query('select 1 as number', function(err, recordset) {
		console.dir(recordset);
	});
	
	// Stored Procedure
	
	var request = new sql.Request(connection);
	request.input('input_parameter', sql.Int, value);
	request.output('output_parameter', sql.Int);
	request.execute('procedure_name', function(err, recordsets, returnValue) {
		console.dir(recordsets);
	});
	
});
```

## Quick Example with one global connection

```javascript
var sql = require('mssql'); 

config = {
	user: '...',
	password: '...',
	server: 'localhost',
	database: '...'
}

sql.connect(config, function(err) {
	
	// Query
	
	var request = new sql.Request();
	request.query('select 1 as number', function(err, recordset) {
		console.dir(recordset);
	});
	
	// Stored Procedure
	
	var request = new sql.Request();
	request.input('input_parameter', sql.Int, value);
	request.output('output_parameter', sql.Int);
	request.execute('procedure_name', function(err, recordsets, returnValue) {
		console.dir(recordsets);
	});
	
});
```

## Documentation

### Configuration

* [Basic](#cfg-basic)
* [Tedious](#cfg-tedious)
* [Microsoft Driver for Node.js for SQL Server](#cfg-msnodesql)

### Connection

* [connect](#connect)
* [close](#close)

### Request

* [execute](#execute)
* [input](#input)
* [output](#output)
* [query](#query)

### Other

* [Metadata](#meta)
* [Data Types](#data-types)
* [Verbose Mode](#verbose)

## Configuration

```javascript
config = { ... }
```

<a name="cfg-basic" />
### Basic configuration is same for all drivers.

* **driver** - Driver to use (default: `tedious`). Possible values: `tedious` or `msnodesql`.
* **user** - User name to use for authentication.
* **password** - Password to use for authentication.
* **server** - Hostname to connect to.
* **port** - Port to connect to (default: `1433`).
* **database** - Database to connect to (default: dependent on server configuration).

<a name="cfg-tedious" />
### Tedious

* **options** - Tedious specific options. More information: http://pekim.github.io/tedious/api-connection.html
* **pool.max** - The maximum number of connections there can be in the pool (default: `10`).
* **pool.min** - The minimun of connections there can be in the pool (default: `0`).
* **pool.idleTimeoutMillis** - The Number of milliseconds before closing an unused connection (default: `30000`).

__Example__

```javascript
config = {
	options: {
		// tedious options
	},
	
	pool: {
		max: 10,
	    min: 0,
	    idleTimeoutMillis: 30000
	}
}
```

<a name="cfg-msnodesql" />
### Microsoft Driver for Node.js for SQL Server

This driver is not part of the default package and you must install it separately.

* **connectionString** - Connection string (default: `Driver={SQL Server Native Client 11.0};Server=#{server},#{port};Database=#{database};Uid=#{user};Pwd=#{password};`).

## Connection

```javascript
var connection = new sql.Connection({ /* config */ });
```

<a name="connect" />
### connect(callback)

Create connection to the server.

__Arguments__

* **callback(err)** - A callback which is called after connection has established, or an error has occurred.

__Example__

```javascript
var connection = new sql.Connection({
	user: '...',
	password: '...',
	server: 'localhost',
	database: '...'
});

connection.connect(function(err) {
	// ...
});
```

---------------------------------------

<a name="close" />
### close()

Close connection to the server.

__Example__

```javascript
connection.close();
```

## Request

```javascript
var request = new sql.Request(/* [connection] */);
```

If you ommit connection argument, global connection is used instead.

<a name="execute" />
### execute(procedure, [callback])

Call a stored procedure.

__Arguments__

* **procedure** - Name of the stored procedure to be executed.
* **callback(err, recordsets, returnValue)** - A callback which is called after execution has completed, or an error has occurred.

__Example__

```javascript
var request = new sql.Request();
request.input('input_parameter', sql.Int, value);
request.output('output_parameter', sql.Int);
request.execute('procedure_name', function(err, recordsets, returnValue) {
	console.log(recordsets.length); // count of recordsets returned by procedure
	console.log(recordset[0].length); // count of rows contained in first recordset
	console.log(returnValue); // procedure return value
	
	console.log(request.parameters.output_parameter.value); // output value
	
	// ...
});
```

---------------------------------------

<a name="input" />
### input(name, [type], value)

Add an input parameter to the request.

__Arguments__

* **name** - Name of the input parameter without @ char.
* **type** - SQL data type of input parameter. If you omit type, module automaticaly decide which SQL data type should be used based on JS data type.
* **value** - Input parameter value. `undefined` ans `NaN` values are automatically converted to `null` values.

__Example__

```javascript
request.input('input_parameter', value);
request.input('input_parameter', sql.Int, value);
```

__JS Data Type To SQL Data Type Map__

* `String` -> `sql.VarChar`
* `Number` -> `sql.Int`
* `Boolean` -> `sql.Bit`
* `Date` -> `sql.DateTime`

Default data type for unknown object is `sql.VarChar`.

You can define you own type map.

```javascript
sql.map.register(MyClass, sql.Text);
```

You can also overwrite default type map.

```javascript
sql.map.register(Number, sql.BigInt);
```

---------------------------------------

<a name="output" />
### output(name, type)

Add an output parameter to the request.

__Arguments__

* **name** - Name of the output parameter without @ char.
* **type** - SQL data type of output parameter.

__Example__

```javascript
request.output('output_parameter', sql.Int);
```

---------------------------------------

<a name="query" />
### query(command, [callback])

Execute the SQL command.

__Arguments__

* **command** - T-SQL command to be executed.
* **callback(err, recordset)** - A callback which is called after execution has completed, or an error has occurred.

__Example__

```javascript
var request = new sql.Request();
request.query('select 1 as number', function(err, recordset) {
	console.log(recordset[0].number); // return 1
	
	// ...
});
```

You can enable multiple recordsets in querries by `request.multiple = true` command.

```javascript
var request = new sql.Request();
request.multiple = true;

request.query('select 1 as number; select 2 as number', function(err, recordsets) {
	console.log(recordsets[0][0].number); // return 1
	console.log(recordsets[1][0].number); // return 2
	
	// ...
});
```

<a name="data-types" />
## Metadata

Recordset metadata are accessible trough `recordset.columns` property.

```javascript
var request = new sql.Request();
request.query('select 1 as first, \'asdf\' as second', function(err, recordset) {
	console.dir(recordset.columns);
	
	console.log(recordset.columns.first.type === sql.Int); // true
	console.log(recordset.columns.second.type === sql.VarChar); // true
});
```

Columns structure for example above:

```
{ first: { name: 'first', size: 10, type: { name: 'int' } },
  second: { name: 'second', size: 4, type: { name: 'varchar' } } }
```

<a name="data-types" />
## Data Types

```
sql.BigInt
sql.Decimal
sql.Float
sql.Int
sql.Money
sql.Numeric
sql.SmallInt
sql.SmallMoney
sql.Real
sql.TinyInt

sql.Char
sql.NChar
sql.Text
sql.NText
sql.VarChar
sql.NVarChar
sql.Xml

sql.Date
sql.DateTime
sql.DateTimeOffset
sql.SmallDateTime

sql.Bit
sql.UniqueIdentifier
```

Binary types as input parameters are only available with Microsoft's native driver.

```
sql.VarBinary
sql.NVarBinary
sql.Image
```

<a name="verbose" />
## Verbose Mode

You can enable verbose mode by `request.verbose = true` command.

```javascript
var request = new sql.Request();
request.verbose = true;
request.input('username', 'patriksimek');
request.input('password', 'dontuseplaintextpassword');
request.input('attempts', 2);
request.execute('my_stored_procedure');
```

Output for example above could look similar to this.

```
---------- sql execute --------
     proc: my_stored_procedure
    input: @username, varchar, patriksimek
    input: @password, varchar, dontuseplaintextpassword
    input: @attempts, bigint, 2
---------- response -----------
{ id: 1,
  username: 'patriksimek',
  password: 'dontuseplaintextpassword',
  email: null,
  language: 'en',
  attempts: 2 }
---------- --------------------
   return: 0
 duration: 5ms
---------- completed ----------
```

## TODO

* UniqueIdentifier testing.
* Binary, VarBinary, Image testing.

<a name="license" />
## License

Copyright (c) 2013 Patrik Simek

The MIT License

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
