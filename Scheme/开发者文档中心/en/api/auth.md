<script setup>
const loginRequestParams = [
  { name: 'username', type: 'string', required: true, description: 'Username', default: '-' },
  { name: 'password', type: 'string', required: true, description: 'Password', default: '-' }
]

const loginResponseParams = [
  { name: 'token', type: 'string', description: 'Access Token (JWT)' },
  { name: 'expiresIn', type: 'number', description: 'Expiration Time (seconds)' }
]

const userInfoResponseParams = [
  { name: 'id', type: 'number', description: 'User ID' },
  { name: 'username', type: 'string', description: 'Username' },
  { name: 'email', type: 'string', description: 'Email Address' },
  { name: 'roles', type: 'string[]', description: 'User Roles List' },
  { 
    name: 'profile', 
    type: 'object', 
    description: 'Profile',
    subProps: [
        { name: 'avatar', type: 'string', description: 'Avatar URL' },
        { name: 'nickname', type: 'string', description: 'Nickname' }
    ]
  }
]

const columns = {
  name: 'Parameter',
  type: 'Type',
  default: 'Default',
  description: 'Description'
}
</script>

# Authentication

Authentication uses the API-KEY method. Please contact the R&D project team to obtain the KEY.

## 1. Authentication Method

Add API-KEY to the Header.

### Request Headers

| Parameter | Required | Type | Description |
| :--- | :--- | :--- | :--- |
| API-KEY | Yes | string | `<key>` |

:::
