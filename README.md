# Axios에서 プロキシ 설정하기

[![Promo](https://github.com/luminati-io/Rotating-Residential-Proxies/blob/main/50%25%20off%20promo.png)](https://brightdata.co.kr/proxy-types/residential-proxies) 

이 Axios プロキシ 가이드는 다음 주제를 다룹니다:

1. [Axios와 プロキシ](#axios-and-proxies)
2. [Axios에서 プロキ시 사용하기](#using-a-proxy-in-axios)
   - [HTTP/HTTPS プロキシ](#httphttps-proxies)
   - [SOCKS プロ키시](#socks-proxies)
3. [Axios プロ키시: 고급 사용 사례](#axios-proxy-advanced-use-cases)
   - [グローバル로 プロ키시 설정하기](#setting-a-proxy-globally)
   - [Axios에서 プロ키시 인증 처리하기](#dealing-with-proxy-authentication-in-axios)
   - [환경 변수로 プロ키시 설정하기](#setting-proxies-via-environment-variables)
   - [ローテーティングプロキシ 구현하기](#implementing-rotating-proxies)
4. [결론](#conclusion)

## Axios와 プロ키시

[Axios](https://axios-http.com/)는 JavaScript 에코시스템에서 가장 널리 사용되는 HTTP 클라이언트 중 하나입니다. Promise 기반의 사용하기 쉽고 직관적인 API를 제공하여 HTTP リクエスト를 수행하고 커스텀 ヘッダー, 구성, Cookie를 처리할 수 있습니다.

Axios リクエスト를 プロ키시를 통해 라우팅하면 IPアドレス를 마스킹할 수 있어 대상 서버가 사용자를 식별하고 차단하기가 더 어려워집니다.

## Axios에서 プロ키시 사용하기

Axios에서 HTTP, HTTPS 또는 SOCKS プロ키시를 설정해 보겠습니다. `axios` npm 패키지를 설치합니다:

```bash
npm install axios
```

Node.js에서 Axios는 [`proxy`](https://github.com/axios/axios#request-config) config를 통해 HTTP 및 HTTPS プロ키시를 기본으로 지원합니다. 따라서 Node.js 애플리케이션에서 Axios로 HTTP/HTTPS プロ키시를 사용하려면 여기서 추가로 할 일은 없습니다.

반대로 HTTP/S가 아닌 プロ키시를 사용하려면 [Proxy Agents](https://github.com/TooTallNate/proxy-agents) 프로젝트에 의존해야 합니다. 이는 다양한 프로토콜의 プロ키시와 Axios를 통합하기 위한 `http.Agent` 구현을 제공합니다:

- HTTP 및 HTTPS プロ키시: [`https-proxy-agent`](https://github.com/TooTallNate/proxy-agents/blob/main/packages/https-proxy-agent)
- SOCKS, SOCKS5, 및 SOCKS4: [`socks-proxy-agent`](https://github.com/TooTallNate/proxy-agents/blob/main/packages/socks-proxy-agent)
- PAC-\*: [`pac-proxy-agent`](https://github.com/TooTallNate/proxy-agents/blob/main/packages/pac-proxy-agent)

### HTTP/HTTPS プロ키시

HTTP/HTTPS プロ키시의 URL은 다음과 같은 형태여야 합니다:

```
"<PROXY_PROTOCOL>://<PROXY_HOST>:<PROXY_PORT>"
```

- `<PROXY_PROTOCOL>`은 HTTP プロ키시의 경우 “http”, HTTPS プロ키시의 경우 “https”입니다.
- `<PROXY_HOST>`는 일반적으로 원시 IP입니다.
- `<PROXY_PORT>`는 プロ키시 서버가 수신 대기하는 포트입니다.

예를 들어, 다음이 HTTP プロ키시의 URL이라고 가정하겠습니다:

```
"http://47.88.62.42:80"
```

다음과 같이 Axios에서 이 プロ키시를 설정할 수 있습니다:

```js
axios.get(targetURL, {

    proxy: { 

        protocol: "http", 

        host: "47.88.62.42",

        port: 80

    }

})
```

위의 Axios プロ키시 접근 방식이 동작하는지 확인하려면, 무료 HTTP 또는 HTTPS プロ키시 서버의 URL을 가져오면 됩니다. 다음 예시를 시도해 보십시오:

```
Protocol: HTTP; IP Address: 52.117.157.155; Port: 8002
```

완전한 プロ키시 URL은 `http://52.117.157.155:8002`가 됩니다.

プロ키시가 예상대로 동작하는지 확인하려면 HTTPBin 프로젝트의 [/ip](https://httpbin.io/ip) エンドポイント를 대상으로 하십시오. 이 공개 API는 유입 リクエスト의 IP를 반환하므로, プロ키시 서버의 IP를 반환해야 합니다.

Node.js 스크립트의 스니펫은 다음과 같습니다:

```js
import axios from "axios"

async function testProxy() {

    // perform the desired request through the HTTP proxy

const response = await axios.get("https://httpbin.io/ip", {
    proxy: {  
        protocol: "http",  
        host: "52.117.157.155",
        port: 8002
    }
});

    // print the result

    console.log(response.data)

}

testProxy()
```

스크립트를 실행하면 다음이 출력되어야 합니다:

```js
{ "origin": "52.117.157.155" }
```

> **Warning**:\
> 스크립트를 실행하더라도 동일한 결과를 얻지 못할 수 있습니다. 무료 プロ키시 서비스는 신뢰할 수 없고, 느리며, 오류가 잦고, 데이터에 집착하며, 수명이 짧기 때문입니다.

### SOCKS プロ키시

프로키시 config 객체의 protocol 필드에 “socks” 문자열을 설정하려고 하면 다음과 같은 오류가 발생합니다:

```js
AssertionError [ERR_ASSERTION]: protocol mismatch

  // ...

 {

  generatedMessage: false,

  code: 'ERR_ASSERTION',

  actual: 'dada:',

  expected: 'http:',

  operator: '=='

}
```

이는 Axios가 SOCKS プロ키시를 기본으로 지원하지 않기 때문입니다. 프로젝트 의존성에 `socks-proxy-agent` npm 라이브러리를 추가하십시오:

```bash
npm install socks-proxy-agent
```

이 패키지는 Axios에서 HTTP 또는 HTTPS リクエ스트를 수행하면서 SOCKS プロ키시 서버에 연결할 수 있도록 해줍니다.

그다음, 라이브러리에서 SOCKS プロ키시 agent 구현을 import합니다:

```js
const SocksProxyAgent = require("socks-proxy-agent")
```

또는 ESM 사용자라면 다음과 같습니다:

```js
import { SocksProxyAgent } from "socks-proxy-agent"
```

다음이 SOCKS プロ키시의 URL이라고 가정하겠습니다:

```
"socks://183.88.74.73:4153"
```

> **Note**:\
> プロ키시 프로토콜은 “socks”, “socks5”, 또는 “socks4”일 수 있습니다.

이를 변수에 저장하고 `SocksProxyAgent` 생성자에 전달합니다:

```js
const proxyURL = "socks://183.88.74.73:4153"

const proxyAgent = new SocksProxyAgent(proxyURL)
```

`SocksProxyAgent()`는 プロ키시 URL을 통해 HTTP/HTTPS リクエ스트를 수행하기 위한 `http.Agent` 인스턴스를 초기화합니다.

이제 다음과 같이 Axios에서 SOCKS プロ키시를 사용할 수 있습니다:

```js
axios.get(targetURL, { 

    httpAgent: proxyAgent,     

    httpsAgent: proxyAgent 

})
```

`httpAgent`와 `httpsAgent`는 각각 HTTP 및 HTTPS リクエスト를 수행할 때 사용할 커스텀 agent를 정의합니다. 즉, Axios가 수행하는 HTTP 또는 HTTPS リクエ스트는 지정한 SOCKS プロ키시를 통해 전달됩니다. 같은 방식으로 [`https-proxy-agent`](https://www.npmjs.com/package/https-proxy-agent) npm 패키지를 사용하면 Axios에서 HTTP/HTTPS プロ키시를 설정하는 대체 방법으로 사용할 수도 있습니다.

모두 합치면 다음과 같습니다:

```js
import axios from "axios"

import { SocksProxyAgent } from "socks-proxy-agent"

async function testProxy() {

    // replace with the URL of your SOCKS proxy 

    const proxyURL = "socks://183.88.74.73:4153"

    // define the HTTP/HTTPS proxy agent

    const proxyAgent = new SocksProxyAgent(proxyURL)

    // perform the request via the SOCKS proxy

    const response = await axios.get("https://httpbin.io/ip", { 

        httpAgent: proxyAgent,     

        httpsAgent: proxyAgent 

    })

    // print the result

    console.log(response.data) // { "origin": "183.88.74.73" }

}

testProxy()
```

다른 예시는 [how to configure a SOCKS proxy in Axios](https://writech.run/blog/how-to-use-a-socks-proxy-in-axios-6c0355a2e013/) 링크를 참고하십시오.

## Axios プロ키시: 고급 사용 사례

[![Promo](https://github.com/luminati-io/LinkedIn-Scraper/blob/main/Proxies%20and%20scrapers%20GitHub%20bonus%20banner.png)](https://brightdata.co.kr/proxy-types/residential-proxies) 

### グローバル로 プロ키시 설정하기

Axios 인스턴스에 직접 지정하여 プロ키시를 グローバル로 설정할 수 있습니다:

```js
const axiosInstance = axios.create({

    proxy: { 

        protocol: "<PROXY_PROTOCOL>", 

        host: "<PROXY_HOST>",

        port: "<PROXY_PORT>" 

    },

    // other configs...

})
```

또는 Proxy Agents 사용자라면 다음과 같습니다:

```js
// proxy Agent definition ...

const axiosInstance = axios.create({

    httpAgent: proxyAgent,     

    httpsAgent: proxyAgent 

})
```

다음은 Axios가 SOCKS プロ키시를 グローバル로 사용하도록 구성하는 방법입니다:

```js
import { SocksProxyAgent } from "socks-proxy-agent";

const proxyURL = "socks://183.88.74.73:4153";

// Create a SOCKS proxy agent
const proxyAgent = new SocksProxyAgent(proxyURL);

// Create an Axios instance with the SOCKS proxy
const axiosInstance = axios.create({
    httpAgent: proxyAgent, // for HTTP requests
    httpsAgent: proxyAgent, // for HTTPS requests
    // other configs...
});
```

이제 `axiosInstance`로 수행되는 모든 リクエ스트는 자동으로 지정된 プロ키시를 통해 전송됩니다.

### Axios에서 プロ키시 인증 처리하기

프리미엄 プロ키시에 결제 사용자만 접근할 수 있도록, プロ키시 제공업체는 認証으로 이를 보호합니다. 사용자 이름과 비밀번호 없이 인증된 プロ키시에 연결하려고 하면 [407 Proxy Authentication Required](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/407) 오류가 발생합니다.

특히, 인증된 プロ키시의 URL 문법은 다음과 같습니다:

```
[<PROTOCOL>://]<USERNAME>:<PASSWORD>@<HOST>[:<PORT>]
```

예를 들어, 인증된 プロ키시에 연결하기 위한 실제 URL은 다음과 같을 수 있습니다:

```
http://admin:lK4w90MEe45YIkOpk@156.127.0.192:8391
```

이 경우 プロ키시 URL 필드는 다음과 같습니다:

- `<PROTOCOL>:HTTP`
- `<HOST>:156.127.0.192`
- `<PORT>:8391`
- `<USERNAME>:admin`
- `<PASSWORD>:lK4w90MEe45YIkOpk`

Axios에서 プロ키시 인증을 처리하려면, `proxy`의 `authfield`에 사용자 이름과 비밀번호를 지정하십시오:

```js
axios.get(targetURL, {

    proxy: { 

        protocol: "http", 

        host: "156.127.0.192",

        port: "8381",

        auth: {

            username: "admin",

            password: "lK4w90MEe45YIkOpk"

        }

    }

})
```

대신 Proxy Agents 사용자라면 인증을 처리하는 방법이 두 가지 있습니다:

1. プロ키시 URL에 자격 증명을 직접 추가합니다:

```js
var proxyAgent = new SocksProxyAgent("http://admin:[email protected]:8391")
```

2. [URL](https://nodejs.org/api/url.html) 객체에서 `username` 및 `password` 옵션을 설정합니다:

```js
const proxyOpts = new URL("http://156.127.0.192:8391")

proxyOpts.username = "admin"

proxyOpts.password = "lK4w90MEe45YIkOpk"

const proxyAgent = new SocksProxyAgent(proxyOpts)
```

동일한 접근 방식은 HttpsProxyAgent에서도 동작합니다.

### 환경 변수로 プロ키시 설정하기

Axios에서 プロ키시를 グローバル로 구성하는 또 다른 방법은 다음 환경 변수를 설정하는 것입니다:

- `HTTP_PROXY`: HTTP リクエ스트에 사용할 プロ키시 서버의 URL입니다.
- `HTTPS_PROXY`: HTTPS リクエ스트에 사용할 プロ키시 서버의 URL입니다.

Linux 또는 macOS에서는 다음과 같이 설정할 수 있습니다:

```bash
export HTTP_PROXY = "[<PROTOCOL>://]<USERNAME>:<PASSWORD>@<HOST>[:<PORT>]"

export HTTPS_PROXY = "[<PROTOCOL>://]<USERNAME>:<PASSWORD>@<HOST>[:<PORT>]"
```

Axios가 이러한 환경 변수를 감지하면, 認証을 위한 자격 증명을 포함하여 해당 변수에서 プロ키시 설정을 읽어옵니다. Axios가 해당 환경 변수를 무시하도록 하려면 `proxy` 필드를 `false`로 설정하십시오. 또한 プロ키시를 적용하지 않아야 하는 도메인 목록을 콤마로 구분하여 `NO_PROXY` env로 정의할 수도 있다는 점을 유념하십시오.

동일한 메커니즘은 [using proxies in cURL](https://brightdata.co.kr/blog/proxy-101/curl-with-proxies)에서도 동작합니다.

### ローテーティングプロキシ 구현하기

대상 사이트가 プロ키시의 IPアドレス를 차단하는 것을 방지하려면, 수행하는 각 リクエスト가 서로 다른 プロ키시 서버에서 시작되도록 보장하십시오:

1. 각기 다른 プロ키시에 연결하기 위한 정보를 담은 객체 리스트를 정의합니다.
2. 각 リクエスト 전에 プロ키시 객체를 무작위로 선택합니다.
3. 선택된 プロ키시를 Axios에 구성합니다.

위에서 설명한 접근 방식은 Bright Data가 제공하는 [ローテーティングプロキシ](https://brightdata.co.kr/solutions/rotating-proxies)와 같은 신뢰할 수 있는 プロ키시 서버 풀에 접근할 수 있다고 가정합니다.

## 결론

Bright Data는 전 세계 최고의 プロ키시 서버를 운영하며, Fortune 500 기업과 20,000명 이상의 고객에게 서비스를 제공합니다. 전 세계 プロ키시 네트워크는 다음을 포함합니다:

*   [Datacenter proxies](https://brightdata.co.kr/proxy-types/datacenter-proxies) – 770,000개 이상의 データセンタープロキシ IP.
*   [Residential proxies](https://brightdata.co.kr/proxy-types/residential-proxies) – 195개 이상 국가에서 7,200만 개 이상의 レジデンシャルプロキシ IP.
*   [ISP proxies](https://brightdata.co.kr/proxy-types/isp-proxies) – 700,000개 이상의 ISPプロキシ IP.
*   [Mobile proxies](https://brightdata.co.kr/proxy-types/mobile-proxies) – 700만 개 이상의 モバイルプロキシ IP.

지금 [Create a free Bright Data account](https://brightdata.co.kr/#popup-155639) 하여 プロ키시 서버를 사용해 보십시오.