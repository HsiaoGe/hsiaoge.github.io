### ngx_core_module配置启动顺序

1. main

2. ngx_init_cycle
    1. ngx_core_module_t::create_conf
        1. ngx_conf_parse
            1. ngx_command_t::set
    2. ngx_core_module_t::init_conf

---

### http初始化配置调用顺序

1. main
2. ngx_init_cycle
    1. `http`ngx_command_t::set => ngx_http_block
    2. ngx_http_module_t::create_main_conf
    3. ngx_http_module_t::create_srv_conf
    4. ngx_http_module_t::create_srv_conf
    5. ngx_http_module_t::preconfiguration
    6. ngx_conf_parse
        1. ngx_command_t::set
    7. ngx_http_module_t::init_main_conf
    8. ngx_http_merge_servers
      1. ngx_http_module_t::merge_srv_conf
       2. ngx_http_module_t::merge_loc_conf
    9. ngx_http_module_t::postconfiguration

---

### 注册handler和filter



