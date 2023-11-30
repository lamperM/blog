Dwarf格式的调试信息存储在以下几个section中:
```
.eh_frame         用于实现栈回溯的信息
.debug_aranges    记录内存地址到编译单元Cu的映射关系
.debug_info       包含DWARF核心DIEs的数据
.debug_abbrev     记录.debug_info段中用到的缩写(Abbreviation)
.debug_line       包含行号信息
.debug_frame      记录Call Frame Infomation
.debug_str        .debug_info中的字符串表
```


需要注意的是这些所有.debug_*开头的节区都是非load的section，
而.eh_frame是loadable属性。


.eh_frame的生成和其他.debug_*是独立的，可以在编译时独立选择。