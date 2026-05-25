<script setup>
const loginRequestParams = [
  { name: 'username', type: 'string', required: true, description: '用户名', default: '-' },
  { name: 'password', type: 'string', required: true, description: '用户密码', default: '-' }
]

const loginResponseParams = [
  { name: 'token', type: 'string', description: '访问令牌 (JWT)' },
  { name: 'expiresIn', type: 'number', description: '过期时间 (秒)' }
]

const userInfoResponseParams = [
  { name: 'id', type: 'number', description: '用户 ID' },
  { name: 'username', type: 'string', description: '用户名' },
  { name: 'email', type: 'string', description: '邮箱地址' },
  { name: 'roles', type: 'string[]', description: '用户角色列表' },
  { 
    name: 'profile', 
    type: 'object', 
    description: '个人资料',
    subProps: [
        { name: 'avatar', type: 'string', description: '头像 URL' },
        { name: 'nickname', type: 'string', description: '昵称' }
    ]
  }
]

const columns = {
  name: '参数名',
  type: '类型',
  default: '默认值',
  description: '说明'
}
</script>

# 认证模块 (Authentication)

认证使用API-KEY方式，KEY获取请联系研发项目组。

## 1. 认证方式

在Header中添加API-KEY


### 请求头

| 参数名 | 必选 | 类型 | 说明 |
| :--- | :--- | :--- | :--- |
| API-KEY | 是 | string | `<key>` |



:::
