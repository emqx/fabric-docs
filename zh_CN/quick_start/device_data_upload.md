# 设备端数据上报

## 物模型数据上报

- 修改 thing_model_demo.c 的三元组信息为设备对应的三元组信息

```c
char *product_key = "KS78JR0J";
char *device_name = "testdemo";
char *secret = "5b28b44337b53ef5261d6335af5c8075";
```

- 修改 host 和 port

```C
char *host = "8.136.220.253";
int port = 1883;
```

- 设置重连间隔和重连次数

```C
int reconnect_interval = 10;
int reconnect_time = 3;
```

- 根据已定义好的物模型所对应 TSL 修改 model_id

```c
char *model_id = "test_model_id";
```

- 根据获取的物模型的 TSL 更改下列参数。

  属性参数修改，根据 web 端生成的物模型描述，如下是属性相关参数，以及相关设置。

```json
"properties": [
    {
      "identifier": "property_a1",
      "is_required": false,
      "data_type": {
        "type": "int"
      }
    },
    {
      "identifier": "property_a2",
      "is_required": false,
      "data_type": {
        "type": "double"
      }
    },
    {
      "identifier": "property_a3",
      "is_required": false,
      "data_type": {
        "type": "text"
      }
    },
    {
      "identifier": "property_b",
      "is_required": false,
      "data_type": {
        "type": "struct",
        "specs": [
          {
            "identifier": "aaaa",
            "data_type": {
              "type": "int"
            }
          },
          {
            "identifier": "bbbb",
            "data_type": {
              "type": "int"
            }
          },
          {
            "identifier": "cccc",
            "data_type": {
              "type": "double"
            }
          }
        ]
      }
    },
    {
      "identifier": "property_c",
      "is_required": false,
      "data_type": {
        "type": "array",
        "specs": {
          "size": 10,
          "item": {
            "type": "int"
          }
        }
      }
    },
    {
      "identifier": "property_d",
      "is_required": false,
      "data_type": {
        "type": "array",
        "specs": {
          "size": 10,
          "item": {
            "type": "double"
          }
        }
      }
    },
    {
      "identifier": "property_e",
      "is_required": false,
      "data_type": {
        "type": "array",
        "specs": {
          "size": 10,
          "item": {
            "type": "text"
          }
        }
      }
    },
    {
      "identifier": "property_f",
      "is_required": false,
      "data_type": {
        "type": "array",
        "specs": {
          "size": 10,
          "item": {
            "type": "struct",
            "specs": [
              {
                "identifier": "asdf",
                "data_type": {
                  "type": "int"
                }
              },
              {
                "identifier": "hjkl",
                "data_type": {
                  "type": "int"
                }
              }
            ]
          }
        }
      }
    }
  ]
```

```c
// 原生类型参数设置
int    val_i1     = 0;
int    val_i2     = 0;
char   val_s3[16] = "text";
double val_d4     = 23.7;
double val_d5     = 23.7;
char   val_s6[16] = "1629344092";

// 结构体类型参数设置
emqc_tm_object val_struct[] = {
    { .type = EMQC_TM_INT, "aaaa", .val_i = 0 },
    { .type = EMQC_TM_INT, "bbbb", .val_i = 0 },
    { .type = EMQC_TM_DOUBLE, "cccc", .val_d = 0.0 },
    { .type = EMQC_TM_TERMINATOR, "", .val_o = NULL }
};

// 数组类型参数设置
int *val_arr_int[] = {&val_i1, &val_i2, NULL};
double *val_arr_dou[] = {&val_d4, &val_d5, NULL};
char *val_arr_str[] = {val_s3, val_s6, NULL};

// 结构体数组类型设置
emqc_tm_object val_struct1[] = {
    { EMQC_TM_INT, "asdf", .val_i = 0 },
    { EMQC_TM_INT, "hjkl", .val_i = 0 },
    // 这里是作为结束符必须的
    { EMQC_TM_TERMINATOR, "", .val_o = NULL } 
};

emqc_tm_object val_struct2[] = {
    { EMQC_TM_INT, "asdf", .val_i = 0 },
    { EMQC_TM_INT, "hjkl", .val_i = 0 },
    // 这里是作为结束符必须的
    { EMQC_TM_TERMINATOR, "", .val_o = NULL } 
};

emqc_tm_object *val_arr_struct[] = {
    val_struct1,
    val_struct2,
    NULL // 这里是作为结束符必须的
};

// 最终要上报的数据全部配置到这里
emqc_tm_object property_array[] = {
    { .type = EMQC_TM_INT,  .key = "property_a1", .val_i = 10 },
    { .type = EMQC_TM_DOUBLE,  .key = "property_a2", .val_d = 10.10 },
    { .type = EMQC_TM_STR,  .key = "property_a3", .val_s = val_s6 },
    { .type = EMQC_TM_STRUCT,  .key = "property_b", .val_o = &val_struct },
    { .type = EMQC_TM_ARR_INT,  .key = "property_c", .val_o = &val_arr_int },
    { .type = EMQC_TM_ARR_DOUBLE,  .key = "property_d", .val_o = &val_arr_dou },
    { .type = EMQC_TM_ARR_STR,  .key = "property_e", .val_o = &val_arr_str },
    { .type = EMQC_TM_ARR_STRUCT,  .key = "property_f", .val_o = &val_arr_struct },
    // 这里是作为结束符必须的
    { .type = EMQC_TM_TERMINATOR, .key = "", .val_o = NULL },
};

```

​		属性上报函数，响应函数以及属性设置回调。

```c
// 属性上报
void property_post(emqc_tm *tm)
{
    char sfx[] = "property/post";
    char mtd[] = "thing.property.post";
    char topic[EMQC_TM_TOPIC_LEN];

    json_object *jso = emqc_tm_new_payload(mtd, NULL);
    int          i   = 0;
    do {
        if (property_array[i].type == EMQC_TM_TERMINATOR) {
            break;
        }
        emqc_tm_add_post_param(jso, &property_array[i], model_id);
    } while (++i);

    snprintf(topic, EMQC_TM_TOPIC_LEN, TM_FMT, product_key, device_name, sfx);
    log_info("#PUB: %s", topic);
    log_info("#PAYLOAD: %s", json_object_to_json_string(jso));
    emqc_tm_pub(tm, topic, json_object_to_json_string(jso));
    json_object_put(jso);
    return;
}

// 属性响应回调
void *property_post_reply_cb(void *payload)
{
    emqc_tm_trans_object *obj = (emqc_tm_trans_object*) payload;
    if (obj) {
        switch (obj->type) {
            // 符合物模型协议的数据，从这里获取
            case TM_REP:;
                tm_reply *tr = obj->rep;
                log_info("#RECV: id: %s, code: %d, data: %s", tr->id, tr->code, tr->data);
                break;
            // 自定义数据，从这里获取
            case TM_RAW:
                log_info("#RECV: raw_data: %s", obj->raw);
                break;
            default:
                log_err("Error trans type.");
        }
    }
    return NULL;
}

// 属性设置回调
void *property_set_cb(void *payload)
{
    emqc_tm_trans_object *obj = (emqc_tm_trans_object*) payload;
    char *ret = NULL;
    if (obj) {
        switch (obj->type) {
            case TM_REQ:;// 符合物模型协议的数据，从这里获取
                tm_request *tr = obj->req;
                log_info("#RECV: id: %s, version: %s, method: %s, params: %s", tr->id,
                         tr->version, tr->method, tr->params);
                // 设置操作
                json_object *jso_tmp = json_tokener_parse(tr->params);
                emqc_tm_property_set(jso_tmp, property_array, model_id);

                json_object *jso      = json_object_new_object();
                json_object *jso_data = json_object_new_object();
                json_object_object_add(jso, "id", json_object_new_string(tr->id));
                json_object_object_add(jso, "code", json_object_new_int(0));
                json_object_object_add(jso, "data", jso_data);
                ret = zstrdup(json_object_to_json_string(jso));
                json_object_put(jso);
                break;
            case TM_RAW:// 自定义数据，从这里获取
                log_info("#RECV: raw_data: %s", obj->raw);
                break;
            default:
                log_err("Error trans type.");
        }
    }

    return (void *) ret;
}
```



​		事件参数修改，根据 web 端生成的物模型描述，如下是事件相关参数，以及相关设置。

```json
 "events": [
    {
      "identifier": "events",
      "is_required": false,
      "event_type": "EVENT_TYPE_INFO",
      "output_data": [
        {
          "identifier": "param01",
          "data_type": {
            "type": "array",
            "specs": {
              "size": 10,
              "item": {
                "type": "text"
              }
            }
          }
        }
      ]
    }
  ]
```



```c
char val_param01_s1[] = "param011";
char val_param01_s2[] = "param012";
char val_param01_s3[] = "param013";
char val_param01_s4[] = "param014";
char val_param01_s5[] = "param015";
char val_param01_s6[] = "param016";
char val_param01_s7[] = "param017";
char val_param01_s8[] = "param018";
char val_param01_s9[] = "param019";

char *param01[] = {
	val_param01_s1,
	val_param01_s2,
	val_param01_s3,
	val_param01_s4,
	val_param01_s5,
	val_param01_s6,
	val_param01_s7,
	val_param01_s8,
	val_param01_s9,
	NULL
 };

char *event_idents[] = {
	"events", // 如果有多个 identifer 顺序添加，最后一个元素一定是 NULL。
	NULL,
};
emqc_tm_object event_array[] = {
	{ .type = EMQC_TM_ARR_STR, .key = "param01", .val_o = &param01 },
	{ .type = EMQC_TM_TERMINATOR, .key = "", .val_o = NULL },
 };
```

​		服务参数修改，根据 web 端生成的物模型描述，如下是服务相关参数，以及相关设置。

```json
 "actions": [
    {
      "identifier": "service01",
      "is_required": false,
      "input_data_param": [
        {
          "identifier": "input_param",
          "data_type": {
            "type": "int"
          }
        }
      ],
      "output_data_param": [
        {
          "identifier": "output_param",
          "data_type": {
            "type": "int"
          }
        }
      ]
    }
  ]
```



```C
char *service_idents[] = {
	"service01",
	NULL,
};
emqc_tm_object service_array[] = {
	{ .type = EMQC_TM_INT, .key = "output_param", .val_i = 0 },
	{ .type = EMQC_TM_TERMINATOR, .key = "", .val_o = NULL },
 };

```

- 根据需求定义自己的服务，将自己的服务放到service_cb中，并处理服务服务的错误响应，以下仅为事例。

```C
void *service_cb(void *payload)
{
    emqc_tm_trans_object *obj = (emqc_tm_trans_object*) payload;
    char *       ret     = NULL;
    if (obj) {
        switch (obj->type) {
            // 符合物模型协议的数据，从这里获取
            case TM_REQ:;
                tm_request *tr = obj->req;
                log_info("#RECV: id: %s, version: %s, method: %s, params: %s", tr->id,
                         tr->version, tr->method, tr->params);
                json_object *jso_tmp = json_tokener_parse(tr->params);
                json_object *jso_val = NULL;
                int          val     = 0;

                int i = 0;
                int j = 0;
                while (service_idents[i] != NULL) {
                    if (!strstr(tr->method, service_idents[i])) {
                        do {
                            if (service_array[j].type == EMQC_TM_TERMINATOR) {
                                break;
                            }

                        } while (++j);

                    } else {
                        json_object *jso      = json_object_new_object();
                        json_object *jso_data = json_object_new_object();
                        do {
                            if (service_array[j].type == EMQC_TM_TERMINATOR) {
                                break;
                            }

                            // 这里可以加具体的操作。
                            // 根据输入计算输出。
                            emqc_tm_add_service_param(jso_data, service_array[j].key,
                                                      &service_array[j]);
                        } while (++j);

                        json_object_object_add(jso, "id", json_object_new_string(tr->id));
                        json_object_object_add(jso, "code", json_object_new_int(0));
                        json_object_object_add(jso, "data", jso_data);
                        ret = zstrdup(json_object_to_json_string(jso));
                        json_object_put(jso);
                        break;
                    }
                    i++;
                    j++;
                }
                break;
            // 自定义数据，从这里获取
            case TM_RAW:
                log_info("#RECV: raw_data: %s", obj->raw);
                break;
            default:
                log_err("Error trans type.");
        }
    }
    return (void *) ret;
}Z
```

- 设备属性获取并设置相应的回调。

```c
// 获取期望属性函数，可在设备启动时，或重连是调用
void get_desire(emqc_tm *tm) {
  // 修改希望获取属性的 key
    const char *keys[] = {"key_xxx", "key_xxx", "key_xxx"};
    char        sfx[]  = "property/desired/get";
    char        topic[EMQC_TM_TOPIC_LEN];

    snprintf(topic, EMQC_TM_TOPIC_LEN, TM_FMT, product_key, device_name, sfx);
    log_info("#PUB: %s", topic);
    json_object *jso = emqc_tm_new_payload("thing.property.desired.get", NULL);
  // 修改 key 的个数
    emqc_tm_add_get_desired_param(jso, (const char **) keys, 3, model_id);
    emqc_tm_pub(tm, topic, json_object_to_json_string(jso));
    json_object_put(jso);
    return;
}

// 回调函数
void *desire_get_reply_cb(void *payload)
{
    emqc_tm_trans_object *obj = (emqc_tm_trans_object*) payload;
    if (obj) {
        switch (obj->type) {
            // 符合物模型协议的数据，从这里获取
            case TM_REP:;
                tm_reply *tr = obj->rep;
                log_info("#RECV: id: %s, code: %d, data: %s", tr->id, tr->code, tr->data);
                break;
            // 自定义数据，从这里获取
            case TM_RAW:
                log_info("#RECV: raw_data: %s", obj->raw);
                break;
            default:
                log_err("Error trans type.");
        }
    }
    return NULL;
}

// 设置回调函数
emqc_tm_set_dft_cb(tm, TM_CB_DESIRED_GET_REPLY, desire_get_reply_cb);

```

- 清除设备属性并设置响应的回调。

```C
// 清除期望属性
void clear_desire(emqc_tm *tm) {
    char          topic[EMQC_TM_TOPIC_LEN];
    char          sfx[] = "property/desired/delete";
    del_des_param ddp[] = {{"WF", NULL}, {"Power", "1"}};

    snprintf(topic, EMQC_TM_TOPIC_LEN, TM_FMT, product_key, device_name, sfx);
    log_info("#PUB: %s", topic);
    json_object *jso = emqc_tm_new_payload("thing.property.desired.delete", NULL);
    emqc_tm_add_del_desired_param(jso, ddp, 2, NULL);
    emqc_tm_pub(tm, topic, json_object_to_json_string(jso));
    json_object_put(jso);
    return;
}

// 回调函数
void *desire_delete_reply_cb(void *payload)
{
    emqc_tm_trans_object *obj = (emqc_tm_trans_object*) payload;
    if (obj) {
        switch (obj->type) {
            case TM_REP:;
                tm_reply *tr = obj->rep;
                log_info("#RECV: id: %s, code: %d, data: %s", tr->id, tr->code, tr->data);
                break;
            case TM_RAW:
                log_info("#RECV: raw_data: %s", obj->raw);
                break;
            default:
                log_err("Error trans type.");
        }
    }
    return NULL;
}

// 设置回调函数
emqc_tm_set_dft_cb(tm, TM_CB_DESIRED_DEL_REPLY, desire_delete_reply_cb);

```

- 到此，我们的物模型就修改完毕，重新编译，既可以上报数据了。

## 设备影子数据上报

- 同物模型相同首先要修改三元组信息以及 host 地址和端口号

- 通过 sdk 提供的一些设置函数，设置三元组，以及对应的回调函数

```C
emqc_ds *ds = emqc_ds_new();

emqc_set_secret(ds->handle, secret);
emqc_set_product_key(ds->handle, product_key);
emqc_set_device_name(ds->handle, device_name);
// 设置回调处理函数
emqc_ds_set_get_callback(ds, get_payload_call);
emqc_ds_set_generic_callback(ds, generic_payload_call);
emqc_ds_set_control_callback(ds, control_payload_call);
// 设置回调函数
emqc_ds_set_reconnect(ds, interval, times);

// 初始化已设置的参数
emqc_ds_init(ds, host, port);
rc = emqc_ds_connect(ds, 60, 1);
```

- 设置请求的 body

```C
serv_req *re_update = new_request();
emqc_ds_state st = DESIRED;
char *key = "test"
char *t2[] = {"11","22","33","44","55"};
add_param_arr(re_update, key, t2, STR, 5, st); 
```

可以通过 ``add_param`` 系列函数添加不同类型的数据，如下：

```C
int add_param_null(serv_req *request, const char *key, emqc_ds_state s);
int add_param_int(serv_req *request, const char *key, int val, emqc_ds_state s);
int add_param_str(serv_req *request, const char *key, const char *val, emqc_ds_state s);
int add_param_arr(serv_req *request, const char *key, void *val, TAG type, int len, emqc_ds_state s);
```



- 请求数据构建好以后，我们可以调用对应的方法向服务器发送请求：

```C
emqc_ds_asyn_update(ds, re_update);
```

请求函数包括以下几种；

```C
EMQC_ERR_CODE emqc_ds_asyn_get(emqc_ds *handle);
EMQC_ERR_CODE emqc_ds_asyn_update(emqc_ds *handle, serv_req *resquest);
EMQC_ERR_CODE emqc_ds_asyn_delete(emqc_ds *handle, serv_req *request);
```

- 最后，我们释放资源

```C
emqc_disconnect(ds->handle);
emqc_ds_destroy(ds);
```

