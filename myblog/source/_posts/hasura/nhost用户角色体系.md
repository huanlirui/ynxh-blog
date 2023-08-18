# hasura + nhost 的用户权限体系配置

## 默认角色

user

## 允许配置的角色

在 auth 的 docker-compose.yml 中找到 auth 服务

```

auth:
    image: nhost/hasura-auth
    depends_on:
      - postgres
      - graphql-engine
    restart: always
    volumes:
      - ./emails:/app/email-templates
    environment:
      AUTH_HOST: '0.0.0.0'
      HASURA_GRAPHQL_DATABASE_URL: postgres://postgres:${POSTGRES_PASSWORD:-secretpgpassword}@postgres:5432/postgres
      HASURA_GRAPHQL_GRAPHQL_URL: http://graphql-engine:8080/v1/graphql
      HASURA_GRAPHQL_JWT_SECRET: ${HASURA_GRAPHQL_JWT_SECRET}
      HASURA_GRAPHQL_ADMIN_SECRET: ${HASURA_GRAPHQL_ADMIN_SECRET}
     #AUTH_LOG_LEVEL: info
      AUTH_CLIENT_URL: ${AUTH_CLIENT_URL:-http://localhost:3000}
      AUTH_SMTP_HOST: mailhog
      AUTH_SMTP_PORT: 1025
      AUTH_SMTP_USER: user
      AUTH_SMTP_PASS: password
      AUTH_SMTP_SENDER: mail@example.com
      AUTH_USER_DEFAULT_ALLOWED_ROLES: icbc,user
      AUTH_EMAIL_SIGNIN_EMAIL_VERIFIED_REQUIRED: false
    expose:
      - 4000
    labels:
      - "traefik.enable=true"
      - "traefik.constraint=auth"
      - "traefik.http.middlewares.strip-auth.stripprefix.prefixes=/v1/auth"
      - "traefik.http.routers.auth.rule= PathPrefix(`/v1/auth`)"
      - "traefik.http.routers.auth.middlewares=strip-auth@docker"
      - "traefik.http.routers.auth.entrypoints=web"
```

### 关键点

    ```
     AUTH_USER_DEFAULT_ALLOWED_ROLES: icbc,user
    ```

这行配置将允许你通过 gql 进行用户和角色的绑定。如上，只能配置 icbc 和 user 用户。

### gql 的 api 示例 给某个用户新增角色和删除角色

```

mutation MyMutation {
  insert_auth_user_roles_one(object: {role: "icbc", user_id: "e4d5cbed-2c5f-4eda-a8b4-c5f8fa1aed9e"}) {
    role
    user {
      display_name
      email
      id
    }
  }
  delete_auth_user_roles_by_pk(id: "01b8645a-b173-483a-8d6f-8e9a073a5efd") {
    id
  }
}
```

### 注意，由于集成了 nhost 的体系。nhost 的登录，将会返回如下参数

#### 可以看到roles中只有icbc，数据库中只有这一条数据

```
{
    "session": {
        "accessToken": "eyJhbGciOiJIUzI1NiJ9.eyJodHRwczovL2hhc3VyYS5pby9qd3QvY2xhaW1zIjp7IngtaGFzdXJhLWFsbG93ZWQtcm9sZXMiOlsiaWNiYyIsInVzZXIiXSwieC1oYXN1cmEtZGVmYXVsdC1yb2xlIjoidXNlciIsIngtaGFzdXJhLXVzZXItaWQiOiJlNGQ1Y2JlZC0yYzVmLTRlZGEtYThiNC1jNWY4ZmExYWVkOWUiLCJ4LWhhc3VyYS11c2VyLWlzLWFub255bW91cyI6ImZhbHNlIn0sInN1YiI6ImU0ZDVjYmVkLTJjNWYtNGVkYS1hOGI0LWM1ZjhmYTFhZWQ5ZSIsImlhdCI6MTY5MjM0NzI5OCwiZXhwIjoxNjkyMzQ4MTk4LCJpc3MiOiJoYXN1cmEtYXV0aCJ9.zK3gd96ZzJqCZcVyn8ao27wUOKAzFtEo95INUcj93JU",
        "accessTokenExpiresIn": 900,
        "refreshToken": "4e54af59-ae29-419a-ad00-e71c483a3b0c",
        "refreshTokenId": "029dc55e-c75c-4bca-8cc7-2cbd7037f23f",
        "user": {
            "id": "e4d5cbed-2c5f-4eda-a8b4-c5f8fa1aed9e",
            "createdAt": "2023-06-25T03:51:43.0722+00:00",
            "displayName": "xiaohei",
            "avatarUrl": "https://s.gravatar.com/avatar/fe9ccee80b9643df67feed51c50e7922?r=g&default=blank",
            "locale": "en",
            "email": "xiaohei@bigzhu.net",
            "isAnonymous": false,
            "defaultRole": "user",
            "metadata": {},
            "emailVerified": true,
            "phoneNumber": null,
            "phoneNumberVerified": false,
            "activeMfaType": null,
            "roles": [
                "icbc"
            ]
        }
    },
    "mfa": null
}
```

#### 但是！hasura认定的是jwt中的参数，解析上面的token得到如下

```
{
    "https://hasura.io/jwt/claims": {
        "x-hasura-allowed-roles": ["icbc", "user"],
        "x-hasura-default-role": "user",
        "x-hasura-user-id": "e4d5cbed-2c5f-4eda-a8b4-c5f8fa1aed9e",
        "x-hasura-user-is-anonymous": "false"
    },
    "sub": "e4d5cbed-2c5f-4eda-a8b4-c5f8fa1aed9e",
    "iat": 1692347298,
    "exp": 1692348198,
    "iss": "hasura-auth"
}

```

#### 可以看到 user是默认就有的用户。即使库中删除也没用

#### 因此，在hasura中每张表的permisstion中，user角色的权限是所有用户都有的。如果要使用精准的列权限控制和角色绑定。请不要给user角色配置，而是配置到其他的自定义角色中
