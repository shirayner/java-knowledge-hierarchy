[toc]



# 一、基本概念







# 二、基本用法

## 1.Json文件转JSONObject

```java
        ClassPathResource resource = new ClassPathResource("menu.json");
        String menuString = "";
        try {
            File menuConfigFile = resource.getFile();
            menuString = FileUtils.readFileToString(menuConfigFile, "UTF-8");
            if (StringUtils.isEmpty(menuString)) {
                return null;
            }
        } catch (IOException e) {

            return null;
        }
        JSONObject menuJson = JSONObject.parseObject(menuString);
```



- menu.json

```json
{
    "menuList": [
        {
            "id": 1,
            "code": "tour_resourcemanage",
            "name": "系统管理",
            "href": "",
            "icon": "icon15",
            "corner": "",
            "menuList": [
                {
                    "id": 2,
                    "code": "tour_resource_X",
                    "name": "用户管理",
                    "href": "",
                    "icon": "",
                    "corner": "",
                    "menuList": [
                        {
                            "id": 3,
                            "code": "Pro_ResManage_InventoryOverview",
                            "name": "新增用户",
                            "href": "http://vendor.package.fat29.qa.nt.com/ivbk/vendor/stockSurveyList",
                            "icon": "",
                            "corner": "",
                            "openMode": ""
                        },
                        {
                            "id": 4,
                            "code": "Pro_ResManage_BatchRelateRes",
                            "name": "修改用户",
                            "href": "http://product.package.fat66.qa.nt.com/Package-Product-InputV3/UITools/MassOperation/MassOperation.aspx?module=11425",
                            "icon": "",
                            "corner": "",
                            "openMode": ""
                        }
                    ],
                    "openMode": ""
                }
            ]
        },
        {
            "id": 5,
            "code": "ottd_scenicspot",
            "name": "权限管理",
            "href": "",
            "icon": "icon14",
            "corner": "",
            "menuList": [
                {
                    "id": 6,
                    "code": "ottd_scenicspot_info",
                    "name": "角色维护",
                    "href": "",
                    "icon": "",
                    "corner": "",
                    "menuList": [
                        {
                            "id": 7,
                            "code": "ottd_scenicspot_list",
                            "name": "角色分配",
                            "href": "http://launchpad.ttd.fws.qa.nt.com/ttd-product-poimanagement/scenicspot/scenicspotlist",
                            "icon": "",
                            "corner": "",
                            "openMode": ""
                        }
                    ],
                    "openMode": ""
                },
                {
                    "id": 8,
                    "code": "ottd_saleunit",
                    "name": "资源管理",
                    "href": "",
                    "icon": "",
                    "corner": "",
                    "menuList": [
                        {
                            "id": 9,
                            "code": "ottd_saleunit_level2_list",
                            "name": "资源新增",
                            "href": "http://launchpad.ttd.fws.qa.nt.com/ttd-product-poimanagement/scenicspot/scenicspotsaleunit",
                            "icon": "",
                            "corner": "",
                            "openMode": ""
                        },
                        {
                            "id": 10,
                            "code": "ottd_saleunit_leve1_list",
                            "name": "资源删除",
                            "href": "http://launchpad.ttd.fws.qa.nt.com/ttd-product-poimanagement/saleunit/level1SaleUnitIndex",
                            "icon": "",
                            "corner": "",
                            "openMode": ""
                        }
                    ],
                    "openMode": ""
                }
            ]
        }
    ],
    "toolBarList": [
        {
            "id": 1,
            "code": "toolbar_event_backlog",
            "name": "事件待办",
            "href": "http://vendor.package.fat29.qa.nt.com/tour/order_event/vbkEventList?applicationCode=tour_product&locale=zh-CN",
            "icon": "",
            "corner": "",
            "openMode": "openNew"
        },
        {
            "id": 2,
            "code": "toolbar_IM",
            "name": "在线咨询",
            "href": "http://imvendor.fat44.qa.nt.com/main?currentTab=BC",
            "icon": "",
            "corner": "",
            "openMode": "openNew"
        },
        {
            "id": 3,
            "code": "toolbar_vendorIM",
            "name": "商户咨询",
            "href": "",
            "icon": "",
            "corner": "",
            "openMode": "openNew"
        }
    ]
}
```













