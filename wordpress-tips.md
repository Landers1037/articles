---
title: wordpress屏蔽代码
name: wordpress-tips
date: 2018-03-11
id: 0
tags: [wordpress,脚本]
categories: []
abstract: ""
---


`wordpress的后台模块屏蔽代码 在后台php语言的fuction.php里添加代码`

`屏蔽左侧菜单`
<!--more-->


`wordpress的后台模块屏蔽代码 在后台php语言的fuction.php里添加代码`

`屏蔽左侧菜单`<!--more-->

    function remove_menus() {
        global $menu;
        $restricted = array(
            __('Dashboard'),
            __('Posts'),
            __('Media'),
            __('Links'),
            __('Pages'),
            __('Appearance'),
            __('Tools'),
            __('Users'),
            __('Settings'),
            __('Comments'),
            __('Plugins')
        );
        end ($menu);
        while (prev($menu)){
            $value = explode(' ',$menu[key($menu)][0]);
            if(strpos($value[0], '<') === FALSE) {
                if(in_array($value[0] != NULL ? $value[0]:"" , $restricted)){
                    unset($menu[key($menu)]);
                }
            }else {
            $value2 = explode('<', $value[0]);
                if(in_array($value2[0] != NULL ? $value2[0]:"" , $restricted)){
                    unset($menu[key($menu)]);
                }
            }
        }
    }
    if (is_admin()){
        // 屏蔽左侧菜单
        add_action('admin_menu', 'remove_menus');
    }

    屏蔽子菜单

    function remove_submenu() {
        // 删除”设置”下面的子菜单”隐私”
        remove_submenu_page('options-general.php', 'options-privacy.php');
        // 删除”外观”下面的子菜单”编辑”
        remove_submenu_page('themes.php', 'theme-editor.php');
    }
    if (is_admin()){
        //删除子菜单
        add_action('admin_init','remove_submenu');
    }

屏蔽后台更新

    function wp_hide_nag() {
        remove_action( 'admin_notices', 'update_nag', 3 );
    }
    add_action('admin_menu','wp_hide_nag');

屏蔽后台显示和提示帮助

    function remove_screen_options(){ return false;}
        add_filter('screen_options_show_screen', 'remove_screen_options');
        add_filter( 'contextual_help', 'wpse50723_remove_help', 999, 3 );
        function wpse50723_remove_help($old_help, $screen_id, $screen){
        $screen->remove_help_tabs();
        return $old_help;
    }

屏蔽主页模块

    function example_remove_dashboard_widgets() {
        // Globalize the metaboxes array, this holds all the widgets for wp-admin
        global $wp_meta_boxes;
        // 以下这一行代码将删除 "快速发布" 模块
        unset($wp_meta_boxes['dashboard']['side']['core']['dashboard_quick_press']);
        // 以下这一行代码将删除 "引入链接" 模块
        unset($wp_meta_boxes['dashboard']['normal']['core']['dashboard_incoming_links']);
        // 以下这一行代码将删除 "插件" 模块
        unset($wp_meta_boxes['dashboard']['normal']['core']['dashboard_plugins']);
        // 以下这一行代码将删除 "近期评论" 模块
        unset($wp_meta_boxes['dashboard']['normal']['core']['dashboard_recent_comments']);
        // 以下这一行代码将删除 "近期草稿" 模块
        unset($wp_meta_boxes['dashboard']['side']['core']['dashboard_recent_drafts']);
        // 以下这一行代码将删除 "WordPress 开发日志" 模块
        unset($wp_meta_boxes['dashboard']['side']['core']['dashboard_primary']);
        // 以下这一行代码将删除 "其它 WordPress 新闻" 模块
        unset($wp_meta_boxes['dashboard']['side']['core']['dashboard_secondary']);
        // 以下这一行代码将删除 "概况" 模块
        unset($wp_meta_boxes['dashboard']['normal']['core']['dashboard_right_now']);
    }
    add_action('wp_dashboard_setup', 'example_remove_dashboard_widgets' );

屏蔽后台页脚版本

    function change_footer_admin () {return '';}
    add_filter('admin_footer_text', 'change_footer_admin', 9999);
    function change_footer_version() {return '';}
    add_filter( 'update_footer', 'change_footer_version', 9999);

屏蔽后台wp的logo

    function annointed_admin_bar_remove() {
            global $wp_admin_bar;
            /* Remove their stuff */
            $wp_admin_bar->remove_menu('wp-logo');
    }
    add_action('wp_before_admin_bar_render', 'annointed_admin_bar_remove', 0);