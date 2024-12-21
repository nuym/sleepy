# API

1. [只读接口](#read-only)
2. [Status 接口](#status)
3. [Device status 接口](#device)

## Read-only

[Back to # api](#api)

|                      | 路径           | 方法  | 作用             |
| -------------------- | -------------- | ----- | ---------------- |
|                      | `/`            | `GET` | 显示主页         |
| [Jump](#query)       | `/query`       | `GET` | 获取状态         |
| [Jump](#status-list) | `/status_list` | `GET` | 获取可用状态列表 |

### query

[Back to ## read-only](#read-only)

> `/query`

获取当前的状态

* Method: GET
* 无需鉴权

#### Response

```jsonc
{
    "success": true, // 请求是否成功
    "status": 0, // 获取到的状态码
    "info": { // 对应状态码的信息
        "name": "活着", // 状态名称
        "desc": "目前在线，可以通过任何可用的联系方式联系本人。", // 状态描述
        "color": "awake"// 状态颜色, 对应 static/style.css 中的 .sleeping .awake 等类
    },
    "refresh": 5000 // 刷新时间 (ms)
}
```

### status-list

[Back to ## read-only](#read-only)

> `/status_list`

获取可用状态的列表

* Method: GET
* 无需鉴权
* Alias: `/get/status_list` *(兼容旧版本)*

#### Response

```jsonc
[
    {
        "id": 0, // 索引，取决于配置文件中的有无
        "name": "活着", // 状态名称
        "desc": "目前在线，可以通过任何可用的联系方式联系本人。", // 状态描述
        "color": "awake" // 状态颜色, 对应 static/style.css 中的 .sleeping .awake 等类
    }, 
    {
        "id": 1, 
        "name": "似了", 
        "desc": "睡似了或其他原因不在线，紧急情况请使用电话联系。", 
        "color": "sleeping"
    }, 
    // 以此类推
]
```

> 就是返回 `config.json` 中的 `status_list` 列表

## Status

[Back to # api](#api)

|                 | 路径                                   | 方法  | 作用     |
| --------------- | -------------------------------------- | ----- | -------- |
| [1](status-set) | `/set?secret=<secret>&status=<status>` | `GET` | 设置状态 |
|                 | `/set/<secret>/<status>`               | `GET` | -        |


### status-set

[Back to ## status](#status)

> `/set?secret=<secret>&status=<status>`

设置当前状态

* Method: GET
* Alias: `/set/<secret>/<status>`

#### Params

- `<secret>`: 在 `config.json` 中配置的 `secret`
- `<status>`: 状态码 *(`int`)*

#### Response

```jsonc
// 成功
{
    "success": true, // 请求是否成功
    "code": "OK", // 返回代码
    "set_to": 0 // 设置到的状态码
}

// 失败 - 密钥错误
{
    "success": false, // 请求是否成功
    "code": "not authorized", // 返回代码
    "message": "invaild secret" // 详细信息
}

// 失败 - 请求无效
{
    "success": false, // 请求是否成功
    "code": "bad request", // 返回代码
    "message": "argument 'status' must be a number" // 详细信息
}
```

## Device

[Back to # api](#api)

|                    | 路径                                                                                          | 方法   | 作用                          |
| ------------------ | --------------------------------------------------------------------------------------------- | ------ | ----------------------------- |
| [1](device-set)    | `/device/set`                                                                                 | `POST` | 设置单个设备的状态 (打开应用) |
|                    | `/device/set?secret=<secret>&id=<id>&show_name=<show_name>&using=<using>&app_name=<app_name>` | `GET`  | -                             |
| [2](device-remove) | `/device/remove?secret=<secret>&name=<device_name>`                                           | `GET`  | 移除单个设备的状态            |
| [3](device-clear)  | `/device/clear?secret=<secret>`                                                               | `GET`  | 清除所有设备的状态            |

### device-set

[Back to ## device](#device)

> `/device/set`

设置单个设备的状态

* Method: GET / POST

#### Params (GET)

> [!WARNING]
> 使用 url params 传递参数在某些情况下 *(如内容包含特殊符号)* 可能导致非预期行为, 此处更建议使用 POST

> `/device/set?secret=<secret>&id=<id>&show_name=<show_name>&using=<using>&app_name=<app_name>`

- `<secret>`: 在 `config.json` 中配置的 `secret`
- `<id>`: 设备标识符
- `<show_name>`: 显示名称
- `<using>`: 是否正在使用
- `<app_name>`: 正在使用应用的名称

#### Body (POST)

> `/device/set`

```jsonc
{
    "secret": "MySecretCannotGuess", // 密钥
    "id": "device-1", // 设备标识符
    "show_name": "MyDevice1", // 显示名称
    "using": true, // 是否正在使用
    "app_name": "VSCode" // 正在使用应用的名称
}
```

#### Response

```jsonc
// 成功
{
  "success": true,
  "code": "OK"
}

// 失败 - 密钥错误
{
    "success": false,
    "code": "not authorized",
    "message": "invaild secret"
}

// 失败 - 缺少参数
{
    "success": false,
    "code": "bad request",
    "message": "missing param"
}
```

### device-remove

[Back to ## device](#device)

> `/device/remove?secret=<secret>&id=<device_id>`

移除单个设备的状态

* Method: GET

#### Params

- `<secret>`: 在 `config.json` 中配置的 `secret`
- `<device_id>`: 设备标识符

### Response

```jsonc
// 成功
{
    "success": true,
    "code": "OK"
}

// 失败 - 不存在 (也不算失败了?)
{
    "success": false,
    "code": "not found",
    "message": "cannot find item"
}

// 失败 - 密钥错误
{
    "success": false,
    "code": "not authorized",
    "message": "invaild secret"
}
```

### device-clear

[Back to ## device](#device)

> `/device/clear?secret=<secret>`

清除所有设备的状态

* Method: GET

#### Params

- `<secret>`: 在 `config.json` 中配置的 `secret`

#### Response

```jsonc
// 成功
{
    "success": true,
    "code": "OK"
}

// 失败 - 密钥错误
{
    "success": false,
    "code": "not authorized",
    "message": "invaild secret"
}
```

## Storage

|                                | 路径                             | 方法  | 作用                 |
| ------------------------------ | -------------------------------- | ----- | -------------------- |
| [Jump](#storage-reload-config) | `/reload_config?secret=<secret>` | `GET` | 重载配置             |
| [Jump](#storage-save-data)     | `/save_data?secret=<secret>`     | `GET` | 保存内存中的状态信息 |

### storage-reload-config

[Back to ## storage](#storage)

> `/reload_config?secret=<secret>`

重新从 `config.json` 加载配置

* Method: GET

#### Params

- `<secret>`: 在 `config.json` 中配置的 `secret`

#### Response

```jsonc
// 成功
{
    "success": true,
    "code": "OK",
    "config": { // 你的 config.json 内容
        "version": "2024.12.20.1",
        "debug": true,
        "host": "::",
        "port": 9010,
        // ...
    }
}

// 失败 - 密钥错误
{
    "success": false,
    "code": "not authorized",
    "message": "invaild secret"
}
```

### storage-save-data

[Back to ## storage](#storage)

> `/save_data?secret=<secret>`

保存内存中的状态信息到 `data.json`

* Method: GET

#### Params

- `<secret>`: 在 `config.json` 中配置的 `secret`

#### Response

```jsonc
// 成功
{
    "success": true,
    "code": "OK",
    "data": { // data.json 内容
        "status": 0,
        "device_status": {},
        "last_updated": "2024-12-21 13:58:38"
    }
}

// 失败 - 密钥错误
{
    "success": false,
    "code": "not authorized",
    "message": "invaild secret"
}
```