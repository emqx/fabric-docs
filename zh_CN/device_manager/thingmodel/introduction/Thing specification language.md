#物模型 TSL 字段说明
TSL 是将用户定义的物模型以 JSON 的格式导出，用于描述物模型中的能力信息

*以下示例中包含所有参数类型可能出现的字段，实际导出 TSL 不一定包含这么多字段。参数后的文字为参数说明，非参数值*
```json
{
  "properties": [
    {
      "identifier": "能力唯一标识符",
      "is_required": false,
      "data_type": {
        "type": "int（整型）、float（浮点）、double（浮点）、text（字符串）、date（String类型UTC毫秒）、enum（枚举）、bool(布尔）、struct（结构体）、array（数组）",
        "specs": {
          "min":  "参数最小值（int、float、double类型特有）",
          "max":  "参数最大值（int、float、double类型特有）",
          "unit": "属性单位（int、float、double类型特有）",
          "unitName": "属性单位名称（int、float、double类型特有）",
          "length": "文本长度，范围为1~10240（text类型特有）",
    	  "0": "0值文字描述（bool类型必须包含该值）",
          "1": "1值文字描述（bool类型必须包含该值）"
          "size": "数组元素个数，范围为1~50（array类型特有）",
          "item": {
            "type": "数组元素的类型（array类型特有）"
          }
        }
      }
    }
  ],
  "events": [
    {
      "identifier": "事件唯一标识符",
      "is_required": false,
      "event_type": "事件类型（EVENT_TYPE_INFO、EVENT_TYPE_ALERT、EVENT_TYPE_ERROR）",
      "output_data": [
        {
          "identifier": "事件输出参数的标识符，事件内保持唯一即可",
          "data_type": {
            "type": "事件输出参数的类型"
          }
        }
      ]
    }
  ],
  "actions": [
    {
      "identifier": "方法标识符",
      "is_required": false,
      "input_data_param": [
        {
          "identifier": "方法输入参数标识符，输入内保证唯一",
          "data_type": {
            "type": "入参类型"
          }
        }
      ],
      "output_data_param": [
        {
          "identifier": "方法输出参数标识符，输出参数内保证唯一",
          "data_type": {
            "type": "出参类型"
          }
        }
      ]
    }
  ]
}
