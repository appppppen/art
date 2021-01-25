## nodejs项目，从.env文件加载环境变量，ditenv简介
### 1.简介
dotenv就是一个可以使得Node.js项目，从文件中加载环境变量的NPM包。安装dotenv后，我们只需要将程序的环境变量配置写在.env文件中。其作用将.env配置文件解析为json对象，并对其中的key-value键值对通过process.env将其赋值为环境变量。之后便可通过process.env[key]，process.env.MONGO_URI获取对应环境变量。
### 2.安装
```
# with npm
npm install dotenv

# or with Yarn
yarn add dotenv
```
### 3.使用
- 一般在项目入口文件index.js中，尽早引入dotenv
```javascript
require('dotenv').config();
```
- 在项目的根目录中创建.env文件，在新行中以name=value的形式添加特定于环境的变量，例如：
```
MYSQL_HOST=localhost
MYSQL_USER=root
MYSQL_PASSWORD=xxxxxx
MONGO_URI=user:password@localhost:27017/data_develop?authSource=admin
MONGO_POOLSIZE=5
```
现在process.env拥有您在.env文件中定义的键和值, 例如：
```javascript
const db = require('db')
db.connect({
  host: process.env.MYSQL_HOST,
  username: process.env.MYSQL_USER,
  password: process.env.MYSQL_PASSWORD
})

...

const mongoose = require('mongoose');
const mongoose = require('mongoose');
mongoose.connect(`mongodb://${process.env.MONGO_URI}`, {
  poolSize: process.env.MONGO_POOLSIZE,
  autoIndex: false
});
```
### 4.其他实用选择项
- 如果包含环境变量的文件位于其他位置，则可以指定自定义路径。
- Path，Default: path.resolve(process.cwd(), ‘.env’)
```javascript
const argv = require('yargs').argv;
require('dotenv').config({
  path: argv.env
});
```
### 5.dotenv源码：dotenv/lib/main.js
```javascript
const fs = require('fs')
const path = require('path')

function log (message /*: string */) {
  console.log(`[dotenv][DEBUG] ${message}`)
}

const NEWLINE = '\n'
const RE_INI_KEY_VAL = /^\s*([\w.-]+)\s*=\s*(.*)?\s*$/
const RE_NEWLINES = /\\n/g

// Parses src into an Object
function parse (src /*: string | Buffer */, options /*: ?DotenvParseOptions */) /*: DotenvParseOutput */ {
  const debug = Boolean(options && options.debug)
  const obj = {}

  // convert Buffers before splitting into lines and processing
  src.toString().split(NEWLINE).forEach(function (line, idx) {
    // matching "KEY' and 'VAL' in 'KEY=VAL'
    const keyValueArr = line.match(RE_INI_KEY_VAL)
    // matched?
    if (keyValueArr != null) {
      const key = keyValueArr[1]
      // default undefined or missing values to empty string
      let val = (keyValueArr[2] || '')
      const end = val.length - 1
      const isDoubleQuoted = val[0] === '"' && val[end] === '"'
      const isSingleQuoted = val[0] === "'" && val[end] === "'"

      // if single or double quoted, remove quotes
      if (isSingleQuoted || isDoubleQuoted) {
        val = val.substring(1, end)

        // if double quoted, expand newlines
        if (isDoubleQuoted) {
          val = val.replace(RE_NEWLINES, NEWLINE)
        }
      } else {
        // remove surrounding whitespace
        val = val.trim()
      }

      obj[key] = val
    } else if (debug) {
      log(`did not match key and value when parsing line ${idx + 1}: ${line}`)
    }
  })

  return obj
}

// Populates process.env from .env file
function config (options /*: ?DotenvConfigOptions */) /*: DotenvConfigOutput */ {
  let dotenvPath = path.resolve(process.cwd(), '.env')
  let encoding /*: string */ = 'utf8'
  let debug = false

  if (options) {
    if (options.path != null) {
      dotenvPath = options.path
    }
    if (options.encoding != null) {
      encoding = options.encoding
    }
    if (options.debug != null) {
      debug = true
    }
  }

  try {
    // specifying an encoding returns a string instead of a buffer
    const parsed = parse(fs.readFileSync(dotenvPath, { encoding }), { debug })

    Object.keys(parsed).forEach(function (key) {
      if (!process.env.hasOwnProperty(key)) {
        process.env[key] = parsed[key]
      } else if (debug) {
        log(`"${key}" is already defined in \`process.env\` and will not be overwritten`)
      }
    })

    return { parsed }
  } catch (e) {
    return { error: e }
  }
}

module.exports.config = config
module.exports.parse = parse
```