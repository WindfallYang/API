# API
import requests,json,jsonpath,_pytest,hashlib,base64,time



class crm():

    def hash_change(self,data):

        # m = hashlib.md5()
        # m.update(b'123456')
        # user_id = hashlib.md5(b'123').hexdigest()
        # print(user_id)

        # login_name = b'00074778'
        # id = base64.b64encode(login_name)
        # print(id)
        # data = '00074778'
        base64_data = base64.b64encode(data.encode())
        # base64_data_right = base64_data.decode('utf-8')
        base64_data_right =str(base64_data, 'utf-8')

        # print(base64_data_right)
        return base64_data_right

    def l2_crm_login(self):

        loginName = crm().hash_change('00071848')
        password = crm().hash_change('123456')

        url = "http://l2.weilongshipin.com/ac/ltx-authcomponent/web/auth/login"

        payload = json.dumps({
            "loginName": loginName,
            "password": password,
            "spId": "3971474d0c577a0cfef2bcdf99753585"
        })
        headers = {
            'Content-Type': 'application/json'
        }

        response = requests.request("POST", url, headers=headers, data=payload)
        re_json = response.json()
        token = jsonpath.jsonpath(re_json,'$.obj.token')
        print("l2的token为：",token)
        print(re_json)
        return token

    def l1_crm_login(self):

        loginName = crm().hash_change('supadmin')
        password = crm().hash_change('123456')

        url = "http://l1.weilongshipin.com/ac/ltx-authcomponent/web/auth/adminLogin"

        payload = json.dumps({
            "loginName": loginName,
            "password": password
        })
        headers = {
            'Content-Type': 'application/json'
        }

        response = requests.request("POST", url, headers=headers, data=payload)
        re_json = response.json()
        token = jsonpath.jsonpath(re_json,'$.obj.token')
        # print("l1的token为：",token)
        # print(re_json)
        return token[0]

    def l1_crm_get_user_info(self):

        for i in range(0,50):

            i = i+1
            token = crm().l1_crm_login()

            url = "http://l1.weilongshipin.com/uc/ltx-usercomponent/web/userInfo/getPageUserInfoExtList"

            payload = json.dumps({
                "pageNo": i,
                "pageSize": 500,
                "no": "",
                "name": "",
                "mobile": "",
                "locked": ""
            })
            headers = {
                'Authorization':token ,
                'Content-Type': 'application/json'
            }

            response = requests.request("POST", url, headers=headers, data=payload)
            # re_json = json.dumps(response.json())
            re_json = response.json()
            # print(re_json)
            user_id = jsonpath.jsonpath(re_json, '$.obj.records[*].no')

            if user_id == False :
                print("未获取到数据")
                break
            else :
                time_now = time.strftime('%Y-%m-%d %H:%M:%S', time.localtime())
                print(
                    "第" + str(i) + "次获取数据，当前时间是：" + time_now + "\n本次获取到的数据数量是是：" + str(len(user_id)) + "\n本次获取到的数据内容是：")
                print(user_id)
                time.sleep(3)
                continue

    def l1_crm_get_user_total(self):

        token = crm().l1_crm_login()
        url = "http://l1.weilongshipin.com/uc/ltx-usercomponent/web/userInfo/getPageUserInfoExtList"
        payload = json.dumps({
            "pageNo": 1,"pageSize": 10,"no": "","name": "","mobile": "","locked": ""
        })
        headers = {
            'Authorization': token,
            'Content-Type': 'application/json'
        }
        response = requests.request("POST", url, headers=headers, data=payload)
        re_json = response.json()
        user_total = jsonpath.jsonpath(re_json,'$.obj.total')
        print("目前总账号数为："+str(user_total[0]))




if __name__ == '__main__':

    # 获取token
    # crm().l2_crm_login()
    # crm().l1_crm_login()
    #获取用户信息
    #
    crm().l1_crm_get_user_info()
    #获取总用户数量
    #crm().l1_crm_get_user_total()


