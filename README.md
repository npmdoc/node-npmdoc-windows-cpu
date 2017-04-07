# api documentation for  [windows-cpu (v0.1.4)](https://github.com/KyleRoss/windows-cpu)  [![npm package](https://img.shields.io/npm/v/npmdoc-windows-cpu.svg?style=flat-square)](https://www.npmjs.org/package/npmdoc-windows-cpu) [![travis-ci.org build-status](https://api.travis-ci.org/npmdoc/node-npmdoc-windows-cpu.svg)](https://travis-ci.org/npmdoc/node-npmdoc-windows-cpu)
#### CPU monitoring utilities for Node.js apps on Windows.

[![NPM](https://nodei.co/npm/windows-cpu.png?downloads=true)](https://www.npmjs.com/package/windows-cpu)

[![apidoc](https://npmdoc.github.io/node-npmdoc-windows-cpu/build/screenCapture.buildNpmdoc.browser._2Fhome_2Ftravis_2Fbuild_2Fnpmdoc_2Fnode-npmdoc-windows-cpu_2Ftmp_2Fbuild_2Fapidoc.html.png)](https://npmdoc.github.io/node-npmdoc-windows-cpu/build/apidoc.html)

![npmPackageListing](https://npmdoc.github.io/node-npmdoc-windows-cpu/build/screenCapture.npmPackageListing.svg)

![npmPackageDependencyTree](https://npmdoc.github.io/node-npmdoc-windows-cpu/build/screenCapture.npmPackageDependencyTree.svg)



# package.json

```json

{
    "author": "",
    "bugs": {
        "url": "https://github.com/KyleRoss/windows-cpu/issues"
    },
    "dependencies": {},
    "description": "CPU monitoring utilities for Node.js apps on Windows.",
    "devDependencies": {
        "mocha": "~1.14.0",
        "should": "~2.1.0"
    },
    "directories": {},
    "dist": {
        "shasum": "49835d0894ca0311ba68f3d17b5100d791600d4b",
        "tarball": "https://registry.npmjs.org/windows-cpu/-/windows-cpu-0.1.4.tgz"
    },
    "gitHead": "25e1099bf88a75d0e614d45c669c09927cdb14f7",
    "homepage": "https://github.com/KyleRoss/windows-cpu",
    "keywords": [
        "cpu",
        "process",
        "processor",
        "windows",
        "win",
        "win32",
        "stats",
        "module",
        "load"
    ],
    "license": "MIT",
    "main": "index.js",
    "maintainers": [
        {
            "name": "kyleross",
            "email": "kylerross1324@gmail.com"
        }
    ],
    "name": "windows-cpu",
    "optionalDependencies": {},
    "readme": "ERROR: No README data found!",
    "repository": {
        "type": "git",
        "url": "git+https://github.com/KyleRoss/windows-cpu.git"
    },
    "scripts": {},
    "version": "0.1.4"
}
```



# <a name="apidoc.tableOfContents"></a>[table of contents](#apidoc.tableOfContents)

#### [module windows-cpu](#apidoc.module.windows-cpu)
1.  [function <span class="apidocSignatureSpan">windows-cpu.</span>cpuInfo (cb)](#apidoc.element.windows-cpu.cpuInfo)
1.  [function <span class="apidocSignatureSpan">windows-cpu.</span>findLoad (arg, cb)](#apidoc.element.windows-cpu.findLoad)
1.  [function <span class="apidocSignatureSpan">windows-cpu.</span>nodeLoad (cb)](#apidoc.element.windows-cpu.nodeLoad)
1.  [function <span class="apidocSignatureSpan">windows-cpu.</span>processLoad (cb)](#apidoc.element.windows-cpu.processLoad)
1.  [function <span class="apidocSignatureSpan">windows-cpu.</span>totalLoad (cb)](#apidoc.element.windows-cpu.totalLoad)
1.  [function <span class="apidocSignatureSpan">windows-cpu.</span>totalMemoryUsage (cb)](#apidoc.element.windows-cpu.totalMemoryUsage)



# <a name="apidoc.module.windows-cpu"></a>[module windows-cpu](#apidoc.module.windows-cpu)

#### <a name="apidoc.element.windows-cpu.cpuInfo"></a>[function <span class="apidocSignatureSpan">windows-cpu.</span>cpuInfo (cb)](#apidoc.element.windows-cpu.cpuInfo)
- description and source-code
```javascript
function cpuInfo(cb) {
    if(!isFunction(cb)) cb = emptyFn;
    if(!checkPlatform(cb)) return;

    execFile(wmic, ['cpu', 'get', 'Name'], function (error, res, stderr) {
        if(error !== null || stderr) return cb(error || stderr);

        var cpus = res.match(/[^\r\n]+/g).map(function(v) {
            return v.trim();
        });

        cpus.shift();
        cb(null, cpus);
    });
}
```
- example usage
```shell
...
cpuInfo(cb)
-----------
Gets the name of each processor in the machine.

	var cpu = require('windows-cpu');
	
	// Get listing of processors
	cpu.cpuInfo(function(error, results) {
		if(error) {
		return console.log(error);
		}
		
		// results =>
		// [
		//    'Intel(R) Xeon(R) CPU E5-2609 0 @ 2.40GHz',
...
```

#### <a name="apidoc.element.windows-cpu.findLoad"></a>[function <span class="apidocSignatureSpan">windows-cpu.</span>findLoad (arg, cb)](#apidoc.element.windows-cpu.findLoad)
- description and source-code
```javascript
function findLoad(arg, cb) {
    if(!isFunction(cb)) cb = emptyFn;
    if(!checkPlatform(cb)) return;

    var cmd = "wmic path Win32_PerfFormattedData_PerfProc_Process get Name,PercentProcessorTime,IDProcess | findstr /i /c:" + arg
;
    exec(cmd, function (error, res, stderr) {
        if(error !== null || stderr) return cb(error || stderr);
        if(!res) return cb('Cannot find results for provided arg: ' + arg, { load: 0, results: [] });

        var found = res.replace(/[^\S\n]+/g, ':').replace(/\:\s/g, '|').split('|').filter(function(v) {
            return !!v;
        }).map(function(v) {
            var data = v.split(':');
            return {
                pid: +data[0],
                process: data[1],
                load: +data[2]
            };
        });

        var totalLoad = 0;

        found.forEach(function(obj) {
            totalLoad += obj.load;
        });

        var output = {
            load: totalLoad,
            found: found
        };

        cb(null, output);
    });
}
```
- example usage
```shell
...
findLoad(arg, cb)
---------------
Gets the total load in percent for process(es) by a specific search parameter.

	var cpu = require('windows-cpu');

 // Find the total load for "chrome" processes
 cpu.findLoad('chrome', function(error, results) {
      if(error) {
          return console.log(error);
      }

      // results =>
      // {
      //    load: 8,
...
```

#### <a name="apidoc.element.windows-cpu.nodeLoad"></a>[function <span class="apidocSignatureSpan">windows-cpu.</span>nodeLoad (cb)](#apidoc.element.windows-cpu.nodeLoad)
- description and source-code
```javascript
function nodeLoad(cb) {
    findLoad('node', cb);
}
```
- example usage
```shell
...
nodeLoad(cb)
------------
Gets the total load in percent for all Node.js processes running on the current machine.

	var cpu = require('windows-cpu');
	
	// Get total load for all node processes
	cpu.nodeLoad(function(error, results) {
		if(error) {
		return console.log(error);
		}
		
		// results =>
		// {
		//    load: 20,
...
```

#### <a name="apidoc.element.windows-cpu.processLoad"></a>[function <span class="apidocSignatureSpan">windows-cpu.</span>processLoad (cb)](#apidoc.element.windows-cpu.processLoad)
- description and source-code
```javascript
function processLoad(cb) {
    findLoad(process.pid, cb);
}
```
- example usage
```shell
...
processLoad(cb)
---------------
Gets the total load in percent for all processes running on the current machine per CPU.

	var cpu = require('windows-cpu');
	
	// Get load for current running node process
	cpu.processLoad(function(error, results) {
		if(error) {
		return console.log(error);
		}
		
		// results =>
		// {
		//    load: 10,
...
```

#### <a name="apidoc.element.windows-cpu.totalLoad"></a>[function <span class="apidocSignatureSpan">windows-cpu.</span>totalLoad (cb)](#apidoc.element.windows-cpu.totalLoad)
- description and source-code
```javascript
function totalLoad(cb) {
    if (!isFunction(cb)) cb = emptyFn;
    if (!checkPlatform(cb)) return;

    execFile(wmic, ['cpu', 'get', 'loadpercentage'], function (error, res, stderr) {
        if(error !== null || stderr) return cb(error || stderr);

        var cpus = (res.match(/\d+/g) || []).map(function(x) {
            return +(x.trim());
        });

        cb(null, cpus);
    });
}
```
- example usage
```shell
...
totalLoad(cb)
-------------
Gets the total load in percent for all processes running on the current machine per CPU.

	var cpu = require('windows-cpu');
	
	// Get total load on server for each CPU
	cpu.totalLoad(function(error, results) {
		if(error) {
		return console.log(error);
		}
		
		// results (single cpu in percent) =>
		// [8]
...
```

#### <a name="apidoc.element.windows-cpu.totalMemoryUsage"></a>[function <span class="apidocSignatureSpan">windows-cpu.</span>totalMemoryUsage (cb)](#apidoc.element.windows-cpu.totalMemoryUsage)
- description and source-code
```javascript
function totalMemoryUsage(cb) {
    if (!isFunction(cb)) cb = emptyFn;
    if (!checkPlatform(cb)) return;

    var cmd = "tasklist /FO csv /nh";
    exec(cmd, function (error, res, stderr) {
        if(error !== null || stderr) return cb(error || stderr);
        var results = { usageInKb: 0 , usageInMb: 0 , usageInGb: 0 };

        results.usageInKb = res.match(/[^\r\n]+/g).map(function(v) {
            var amt = +v.split('","')[4].replace(/[^\d]/g, '');
            return (!isNaN(amt) && typeof amt === 'number')? amt : 0;
        }).reduce(function(prev, current) {
            return prev + current;
        });

        results.usageInMb = results.usageInKb / 1024;
        results.usageInGb = results.usageInMb / 1024;

        cb(null, results);
    });
}
```
- example usage
```shell
...
totalMemoryUsage(cb)
--------------------
Gets the total memory usage value in KB , MB and GB .

 var cpu = require('windows-cpu');

 // Get the memory usage
 cpu.totalMemoryUsage(function(error, results) {
      if(error) {
          return console.log(error);
      }

      // results =>
      // {
      //    usageInKb: 3236244,
...
```



# misc
- this document was created with [utility2](https://github.com/kaizhu256/node-utility2)
