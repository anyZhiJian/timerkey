# TimerKey
这是一个基于定时器的按键扫描库
# 特性
- 支持检测短按、长按、双击、连续按下等状态
- 由定时器驱动，仅需50Hz低频就可以完成工作
- 采用面向对象的思想进行封装
- 硬件无关，跨平台
- 无需第三方依赖，所需头文件仅C语言标准库
- C语言设计，可以直接移植

# 如何使用？
## STEP 1
创建一个按键对象，可以选择创建默认按键对象或者自定义按键对象
### 创建默认按键对象
调用`tkey_create_default`函数创建默认按键对象  
默认按键对象的属性如下:  
- 按键扫描的FSM工作在**50Hz**的频率下
- 去抖时间:1 tick(20ms@50Hz)
- 长按时间:25 tick(500ms@50Hz)
- 连续按下时间间隔:15 tick(300ms@50Hz)
- 按键按下时电平:0(低电平)

### 创建自定义按键对象
调用`tkey_create`函数创建自定义按键对象，使用`tkey_init_t`结构体初始化按键对象  
结构体参数如下:  
- event_cb:事件回调函数
- detect_cb:检测回调函数
- user_data:传入回调函数的用户数据
- hold_ticks:长按持续时间
- debounce_ticks:去抖时间
- multi_press_interval_ticks:连续按下间隔时间
- pressed_level:按键按下时电平

## STEP 2
调用`tkey_register_cbs`函数为按键注册事件回调函数和检测回调函数  
## STEP 3
调用`tkey_check_init`函数检测按键初始化状态，返回值如下:  
- 0:初始化完成
- -1:初始化未完成

该函数会检测回调函数的有效性，每次注册回调函数后都必须调用该函数
## STEP 4
初始化完成在对应平台的定时器回调中调用`tkey_handler`函数处理按键的扫描事件
# 回调函数编写
## 检测回调函数
```c
int tkey_detect_cb(void *user_data)
{
    int *pin = (int*)user_data;
    return gpio_read(*pin);
}
```
可以在注册回调函数时通过`user_data`传入需要检测的引脚，这样可以将这个检测回调函数注册在不同的按键对象中
## 事件回调函数
所有的按键事件如下:  
- TKEY_EVENT_PRESS:按下
- TKEY_EVENT_LONG_PRESS:长按
- TKEY_EVENT_RELEASE:释放
- TKEY_EVENT_LONG_RELEASE:长按之后释放
- TKEY_EVENT_MULTI_PRESS:连续按下
- TKEY_EVENT_MULTI_RELEASE:连续按下后释放
- TKEY_EVENT_ALL_PRESS:所有按下的状态
- TKEY_EVENT_ALL_RELEASE:所有释放的状态

不同的事件可以通过`|`与操作来完成多状态检测
```c
void tkey_event_cb(tkey_handle_t key, tkey_event_t event, uint8_t press_count, void *user_data)
{
    switch(key)
    {
        case key1:
        if(event & TKEY_EVENT_ALL_PRESS)
        {
            printf("key1 pressed!\r\n");
            printf("key1 pressed count:%d\r\n", (int)press_count);
            ...
        }
        break;
        case key2:
        if(event & TKEY_EVENT_ALL_RELEASE)
        {
            printf("key2 released!\r\n");
            ...
        }
        break;
        ...
    }
}
```
事件回调函数的参数如下:  
- key:按键对象
- event:事件
- press_count:按下次数
- user_data:用户数据

# PORT
**TimerKey**的代码为C语言设计，支持C++环境，对象的创建默认使用C语言的内存标准库，可以直接移植。如有特殊的内存需求，可以将头文件中`tkey_malloc`和`tkey_free`宏进行替换
