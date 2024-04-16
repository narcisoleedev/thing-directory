## ! This repository was forked from https://github.com/linksmart/thing-directory, I am only making custom changes on the code for academic usage, all credits to the authors.

## Thing Directory custom install

A priori, por convenção esta documentação estará presente em português, entretanto será disponibilizada em inglês ao fim do projeto.

Esse arquivo representa a instalção customizada do Thing-Directory, de acordo com a documentação de instalação padrão presente no README desse repositório, caso ache necessário direi para olhar a documentação dos autores, entretanto explicarei detalhes que não forem claros. Qualquer coisa, consulte a [documentação original](./README.md).

### Pré requisitos

Antes de rodar instale na sua máquina:

- [Go](https://go.dev/dl/)
- [Keycloak](https://www.keycloak.org/downloads) (pela aplicação em si \*necessário baixar o jdk para rodar\*, ou como um conteiner Docker) 
- [Postman](https://www.postman.com/downloads/)

### First steps

Primeiramente, clone esse repositório na sua máquina.

```git clone ```

Após isso, <i>build</i> apartir da root do projeto, execute para rodar, buildar pela configuração padrão do <i>Thing Directory</i>.

``` bash
    go build
    go run . --conf=sample_conf/thing-directory.json
```

Após isso, você deve ser capaz de inicializar a aplicação, feche-a por enquanto.

Perçeba que ele utiliza o arquivo sample_conf como base para rodar inicialmente, entretanto o diretório padrão que a aplicação utiliza é denominado <b>conf</b>.

Ainda na root, execute:

``` bash
    mkdir conf
    cp ./sample_conf/thing-directory.json ./conf
```

Com isso você conseguirá rodar normalmente, utilizando ```go run```, entretanto o servidor está com as configurações todas desabilitadas, explicarei como habilitar cada parte:

### API

Antes de tudo, com a aplicação rodando já conseguimos verificar o funcionamento do Thing Directory, para isso usaremos o <b>Postman</b> para testar os endpoints.

Vale a pena ressaltar que a documentação da API está presente em [documentação](https://linksmart.eu/swagger-ui/dist/?url=https://raw.githubusercontent.com/linksmart/thing-directory/master/apidoc/openapi-spec.yml).

Entretanto, explicitarei alguns detalhes que podem não ser muitos claro. Usaremos o endereço 0.0.0.0 e a porta 8081 como padrão da aplicação.

Alguns os endpoints de <b>Registration API</b>:

ex: 

POST
- http://0.0.0.0:8081/things:

BODY 

```json
{
  "@context": "https://www.w3.org/2019/wot/td/v1",
  "title": "Smart Garden System",
  "description": "A garden system that automates watering based on soil moisture and weather predictions.",
  "properties": {
    "soilMoisture": {
      "type": "integer",
      "forms": [{
        "href": "http://192.168.1.108:8080/soilMoisture"
      }],
      "readOnly": true
    },
    "lastWatered": {
      "type": "string",
      "forms": [{
        "href": "http://192.168.1.108:8080/lastWatered"
      }],
      "readOnly": true
    }
  },
  "actions": {
    "waterPlants": {
      "forms": [{
        "href": "http://192.168.1.108:8080/waterPlants"
      }]
    }
  },
  "events": {
    "wateringCompleted": {
      "data": {"type": "boolean"},
      "forms": [{"href": "http://192.168.1.108:8080/events/wateringCompleted"}]
    }
  },
  "securityDefinitions": {
    "basic_sc": {
      "scheme": "nosec"
    }
  },
  "security": "basic_sc"
}
```

GET/PUT/PATCH/DELETE 
- http://0.0.0.0:8081/things/{id}:

Dependendo do método HTTP, coloque no body nada (GET/DELETE) ou uma nova TD (PUT/PATCH).

Alguns os endpoints de <b>Search API</b>:

GET(JSONPATH)
- http://0.0.0.0:8081/search/jsonpath?query=$[?(@.title=='Fibaro Wall Plug')]

Retorna a TD com base nos parâmetros de procura.

GET(XPATH)
- http://localhost:8081/search/xpath?query=//*[title='Smart Lamp']/properties

Retorna a TD com base nos parâmetros de procura.

Leve em consideração que não habilitamos a autenticação ainda, portanto nada deve ser mandado no header.

### Validation

Para habilitar a validação, baixe o json padrão de validação da W3C em [link](https://www.w3.org/2022/wot/td-schema/v1.1).

Esse json apresenta uma série de padrões que uma Thing Description precisa ter para poder ser validada pelo Thing Directory.

Agora cole na pasta conf.

Com isso, modifique as seguintes linhas do *thing-direcory.json*:

```json
"validation": {
    "jsonSchemas": ["./conf/jsonSchema.json"]
  },
```

Rode a aplicação e veja se receba esse log:

```
Loaded JSON Schemas: [./conf/jsonSchema.json]
```

Agora, caso você insira uma TD fora dos padrões, você receberá resposta 401, caso esteja tudo certo 200 normalmente.

### DNS/SD

Para maiores detalhes aconselha-se olhar a documentação original nessa parte, presente em [DNS-SD registration](https://github.com/linksmart/thing-directory/wiki/Discovery-with-DNS-SD).

Para habilitar o DNS/SD sem interfaces de publicação, modifique o *thing-direcory.json*, dessa maneira.

```json
"dnssd": {
    "publish": {
      "enabled": true,
      "instance": "LinkSmart Thing Directory",
      "domain": "local.",
      "interfaces": []
    }
  },
```

Com isso, você habilitará o DNS/SD na sua rede. Confirme rodando a aplicação e vendo o log:

```log
DNS-SD: registering as "LinkSmart Thing Directory._wot._tcp.local.", subtype: _directory
DNS-SD: publish interfaces not set. Will register to all interfaces with multicast support.
```

### HTTP Config

Para fins de teste, não modifiquei as configurações de HTTP da aplicação, mas pode-se alterar normalmente os parâmetros iniciais de porta, endereço ou outro.

```json
"http": {
    "publicEndpoint": "http://fqdn-of-the-host:8081",
    "bindAddr": "0.0.0.0",
    "bindPort": 8081,
    "tls": {
      "enabled": false,
      "keyFile": "./tls/key.pem",
      "certFile": "./tls/cert.pem"
    },
```

### Autenticação

Essa parte deve-se prestar bastante atenção para habilitar a autenticação normalmente.

Primeiro baixe o keycloak e instale conforme a documentação na maneira que você queira instalar.

Para [*on bare metal*](https://www.keycloak.org/getting-started/getting-started-zip)
*** quando criar o cliente, habilite o cliente mo modo de desenvolvimento protegido***



Após todos os passos abra, o Postman, e no endpoint de sua preferência habilite a autenticação OAuth2.

Para verificar os endpoints providos pelo cliente em questão (que no meu caso é chamado client, mas na documentação do keycloak é chamado myclient), acesse o seguinte endpoint: http://localhost:8080/realms/realm/.well-known/openid-configuration.

Pelos endpoints que recebemos de resposta, temos interesse no de autenticação e de *access token** 

- Auth URL: http://localhost:8080/realms/realm/protocol/openid-connect/auth
- Access Token URL: http://localhost:8080/realms/realm/protocol/openid-connect/token

Pegue também o secret do cliente que você criou pelo realm que você criou (pressupõe-se myrealm) e abra o painel de cliente e pegue o secret e coloque no campo Client ID.

Sua configuração deve ficar parecido com isso:

<img width="600px" src="./public/postman1.png"></img>

Após isso, com o Keycloak rodando, clique em Get Access New Token, faça a autenticação e cole o Access Token no campo solicitado.

Com isso antes de tudo, coloque a configuração de autenticação no jsonSchema.json.

```json
"auth": {
      "enabled": true,
      "provider": "keycloak",
      "providerURL": "http://localhost:8080/realms/realm",
      "clientID": "account",
      "basicEnabled": false,
      "authorization": {
```

Rode a aplicação e o Keycloak para testar a autenticação.

Após isso, se todos os passos foram seguidos, você deve receber a autorização e assim uma resposta 200.

### Deploy

*To be completed*


### Port Forwading

*To be completed*









