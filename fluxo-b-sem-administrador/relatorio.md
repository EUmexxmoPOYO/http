# Relatório — Laboratório de Inspeção HTTP/HTTPS — Fluxo B (sem privilégio administrativo)

> **Como usar este template:** substitua os campos `[...]` pelas suas respostas,
> anexe as capturas de tela na pasta `evidencias/` e referencie-as onde indicado.
> Preserve a formatação markdown (tabelas, blocos de código) para facilitar a correção.
>
> **Observação:** toda a análise prática deste relatório é feita sobre tráfego **HTTP em texto claro**. A análise de HTTPS é teórica, baseada na fundamentação do `readme.md` do repositório.

---

## Identificação

| Campo                                       | Valor                                         |
|---------------------------------------------|-----------------------------------------------|
| Nome                                        | Alexandre Silva João Lourenço                 |
| RA                                          | 0050482413030                                 |
| Disciplina                                  | Redes de Computadores                         |
| Turma                                       | ADS 5°Ciclo Noturno                           |
| Data                                        | 15/05/2026                                    |
| Fluxo                                       | **B — Aluno sem privilégio de administrador** |
| SO utilizado                                | Windows 11                                    |
| Ferramenta de proxy                         | Fiddler Classic per-user                      |
| Navegador(es)                               | Chrome 124                                    |
| HTTPS-First Mode / HTTPS-Only desabilitado? | não                                           |

---

## Atividade 1 — Primeira captura (`http://example.com`)

**Captura de tela:** `evidencias/atv1_sessao.png`

**Request-line enviada:**

```http
GET / HTTP/1.1
```

**Status-line recebida:**

```http
HTTP/1.1 200 OK
```

### Pergunta 1.1
> Quantos cabeçalhos o navegador enviou no request? Liste-os.

**Resposta:**
9 cabeçalhos.

Cabeçalhos:
- Host
- Connection
- Upgrade-Insecure-Requests
- User-Agent
- Accept
- Accept-Encoding
- Accept-Language

### Pergunta 1.2
> Qual foi o `Content-Length` da resposta? Se ele não apareceu, registre `Transfer-Encoding`, versão do protocolo ou outro indício observado. O corpo retornado é HTML, texto puro, JSON ou binário? Como você descobriu?

**Resposta:** Transfer-Encoding: chunked

Também foi identificado: Content-Type: text/html
                         Content-Encoding: gzip

Observando "Content-Type: text/html", é possível concluir que o corpo retornado é HTML.

Além disso, a resposta aparece comprimida porque o servidor utilizou: Content-Encoding: gzip
Por isso o conteúdo exibido no Fiddler aparece como caracteres binários/ilegíveis até ser decodificado.

---

## Atividade 2 — Anatomia de um GET (`http://httpbin.org/get?...`)

**Captura de tela:** `evidencias/atv2_raw.png`

**Request-line completa:**

```http
[colar aqui]
```

**Cabeçalhos-chave capturados:**

| Cabeçalho    | Valor                    |
|--------------|--------------------------|
| `Host`       | httpbin.org              |
| `User-Agent` | Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/147.0.0.0 Safari/537.36                                                    |
| `Accept`     | text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7                       |

**Campos do JSON de resposta:**

```json
{
  "args": {
    "aluno": "alexandre_silva",
    "curso": "redes"
  },

  "headers": {
    "Host": "httpbin.org",
    "User-Agent": "Mozilla/5.0...",
    "Accept": "text/html,application/xhtml+xml..."
  },

  "origin": "201.87.98.203"
}
```

### Pergunta 2.1
> O valor do campo `origin` corresponde a qual elemento da rede? Por que normalmente não é o IP local?

**Resposta:** O campo origin corresponde ao endereço IP público que o servidor enxerga como origem da requisição.
Normalmente ele não mostra o IP local da máquina (como 192.168.x.x ou 10.x.x.x) porque a rede utiliza NAT (Network Address Translation) no roteador. Assim, os dispositivos da rede interna acessam a internet usando um único IP público fornecido pelo provedor.

### Pergunta 2.2
> Compare o `User-Agent` enviado com o que aparece no JSON da resposta. Coincidem?

**Resposta:** Sim. O valor do User-Agent enviado na requisição coincide com o valor retornado no JSON da resposta.

### Pergunta 2.3
> Em `http://httpbin.org/headers`, liste até três cabeçalhos que o servidor vê mas **não aparecem** no Raw do request. De onde vêm? Se não encontrar três, explique por que o resultado pode variar.

**Resposta:**

| Cabeçalho visto pelo servidor | Origem provável               | Observação                                                               |
| ----------------------------- | ----------------------------- | ------------------------------------------------------------------------ |
| `X-Amzn-Trace-Id`             | Infraestrutura da Amazon AWS  | Adicionado automaticamente para rastreamento da requisição               |
| `Via`                         | Proxy/CDN intermediária       | Pode ser inserido por servidores intermediários entre cliente e servidor |
| `X-Forwarded-For`             | Proxy ou balanceador de carga | Usado para informar o IP original do cliente                             |


---

## Atividade 3 — POST e envio de formulário (`http://httpbin.org/forms/post` → `/post`)

**Captura de tela:** `evidencias/atv3_post_raw.png`

**Request-line do POST:**

```http
POST /post HTTP/1.1
```

**Cabeçalhos do request:**

| Cabeçalho        | Valor                               |
| ---------------- | ----------------------------------- |
| `Content-Type`   | `application/x-www-form-urlencoded` |
| `Content-Length` | `190`                               |


**Corpo completo do request:**

```
custname=Alexandre+Silva+Jo%C3%A3o+Louren%C3%A7o&custtel=123456789&custemail=alexandre.silva.joao.05%40gmail.com&size=medium&topping=bacon&topping=onion
```

**Trecho do JSON de resposta (campo `form`):**

```json
"form": {}
```

### Pergunta 3.1
> Qual o formato do corpo? Como esse formato codifica caracteres especiais (espaço, acentos)?

**Resposta:** application/x-www-form-urlencoded

### Pergunta 3.2
> Comparando **Request → WebForms** e **Request → Raw**: qual das duas corresponde literalmente aos bytes enviados no socket TCP?

**Resposta:** Request → WebForms

### Pergunta 3.3 — Composer
> Envie manualmente via Composer um `POST` para `http://httpbin.org/post` com JSON. Registre a resposta. Qual campo do JSON confirma que o servidor interpretou o JSON?

**Captura de tela:** `evidencias/atv3_composer.png`

**Response JSON (trecho relevante):**

```json
{
  [colar aqui]
}
```

**Resposta:** 
data={
  "nome": "Alexandre",
  "idade": 22
}

---

## Atividade 4 — Catálogo de status codes (`http://httpbin.org/...`)

**Captura de tela (lista do Fiddler com as 7 sessões):** `evidencias/atv4_lista.png`

#,Método,URL,Status-line,Content-Length / Transfer-Encoding,Body presente?
1 | GET | .../status/200,HTTP/1.1 200 OK | Content-Length: 0 | Não
2 | GET | .../redirect-to?status_code=301... | HTTP/1.1 301 MOVED PERMANENTLY | Content-Length: 0 | Não
3 | GET | .../status/404 | HTTP/1.1 404 NOT FOUND | Content-Length: 0 | Não
4 | GET | .../status/418 | HTTP/1.1 418 I'M A TEAPOT | Content-Length: 135 | Sim
5 | GET | .../status/500 | HTTP/1.1 500 INTERNAL SERVER ERROR | Content-Length: 0 | Não
6 | GET | .../status/503 | HTTP/1.1 503 SERVICE UNAVAILABLE | Content-Length: 0 | Não
7 | GET | .../cache (+ headers) | HTTP/1.1 304 NOT MODIFIED | (Vazio / Não enviado) | Não

### Pergunta 4.1
> Em qual dos status o corpo está ausente/tamanho zero? Isso é obrigatório pela especificação ou depende do servidor?

**Resposta:** os status 200, 301, 304, 404, 500 e 503 apresentaram corpo tamanho zero. Se for um status 204 ou 304, o corpo deve ser zero. Para os demais (200, 404, 5xx), o tamanho do corpo fica a critério do desenvolvedor ou do servidor web utilizado.

### Pergunta 4.2
> No `301`, qual cabeçalho da resposta informa para onde ir? O que aconteceria se estivesse ausente?

**Resposta:** No campo Response Headers, dentro da seção Transport está o cabeçalho Location: /get.

### Pergunta 4.3
> Diferença semântica entre `200`, `304` e `404` do ponto de vista do cache do navegador.

**Resposta:** 200 OK (Fresh): O navegador entende que o recurso é novo ou que sua versão local expirou. Ele baixa o conteúdo completo do servidor e atualiza o cache com essa nova cópia para usos futuros.

304 Not Modified (Revalidated): O navegador pergunta ao servidor: "Eu tenho a versão X, ela ainda vale?". O servidor responde 304 para dizer que nada mudou. O navegador não baixa o arquivo novamente e usa a cópia que já está no cache, economizando banda.

404 Not Found (Dead/Invalid): O recurso não existe mais no servidor. Para o cache, isso geralmente sinaliza que qualquer cópia antiga armazenada localmente é inválida ou obsoleta, interrompendo a entrega daquele conteúdo.

---

## Atividade 5 — Identificação de cabeçalhos (`http://httpbin.org/response-headers?...` + `/gzip`)

**Captura de tela (Inspectors → Headers):** `evidencias/atv5_headers.png`

| Cabeçalho                    | Req/Resp | Valor capturado | Função em uma frase |
|------------------------------|----------|------------------|----------------------|
| `Host`                       | Req    | httpbin.org            | [...]                |
| `User-Agent`                 | Req    | Mozilla/5.0 (Windows NT 10.0; Win64; x64)...    | [...]                |
| `Accept`                     | Req    | [...]            | [...]                |
| `Accept-Encoding`            | Req    | [...]            | [...]                |
| `Cookie`                     | Req    | [...]            | [...]                |
| `Server`                     | Req    | [...]            | [...]                |
| `Content-Type`               | Req    | [...]            | [...]                |
| `Content-Encoding`           | Req    | [...]            | [...]                |
| `Set-Cookie`                 | Req    | [...]            | [...]                |
| `Cache-Control`              | Req    | [...]            | [...]                |
| `Strict-Transport-Security`  | Não esperado em HTTP — ver Pergunta 5.3 | — | — |

| Cabeçalho                  | Req/Resp | Valor capturado                                            | Função em uma frase                                                                |
|----------------------------|:--------:|------------------------------------------------------------|------------------------------------------------------------------------------------|
| `Host`                     |   Req    | httpbin.org                                                | Indica o domínio do servidor para o qual a requisição está sendo enviada.          |
| `User-Agent`               |   Req    | Mozilla/5.0 (Windows NT 10.0; Win64; x64)...               | Identifica o navegador e o sistema operacional que originaram a requisição.        |
| `Accept`                   |   Req    | text/html,application/xhtml+xml,application/xml;q=0.9...   | Informa ao servidor quais tipos de conteúdo (formatos) o cliente consegue ler.    |
| `Accept-Encoding`          |   Req    | gzip, deflate                                              | Indica os algoritmos de compressão de dados que o cliente suporta receber.         |
| `Cookie`                   |   Req    | teste=1                                                    | Envia ao servidor dados armazenados anteriormente para manter o estado da sessão.  |
| `Server`                   |   Resp   | gunicorn/19.9.0                                            | Identifica o software de servidor web utilizado para processar a requisição.       |
| `Content-Type`             |   Resp   | application/json                                           | Especifica o tipo de mídia (formato) do corpo da mensagem enviada pelo servidor.   |
| `Content-Encoding`          |   Resp   | gzip                                                       | Indica que o corpo da resposta foi compactado usando o algoritmo gzip.             |
| `Set-Cookie`               |   Resp   | teste=1                                                    | Solicita ao navegador que armazene um dado para ser enviado em requisições futuras. |
| `Cache-Control`            |   Resp   | max-age=3600                                               | Define por quanto tempo o navegador deve manter o recurso em cache local.          |
| `Strict-Transport-Security`|   ---    | Não esperado em HTTP — ver Pergunta 5.3                    | —                                                                                  |
### Pergunta 5.1
> `Content-Encoding: gzip`/`br` apareceu? Compare `Content-Length`, quando presente, com o conteúdo visível. O que explica a diferença?

**Resposta:** [...]

### Pergunta 5.2
> Cliente envia `Accept: application/json` mas o recurso só existe em `text/html`. Qual status code esperar?

**Resposta:** [...]

### Pergunta 5.3
> `Strict-Transport-Security` apareceu nas respostas HTTP? Por que esse cabeçalho está ausente neste fluxo? (Consulte a RFC 6797.) Qual é seu papel contra downgrades para HTTP puro?

**Resposta:** [...]

---

## Atividade 6 — HTTP vs HTTPS (análise sem decriptação)

**Captura de tela HTTP (`neverssl.com`):** `evidencias/atv6_http.png`
**Captura de tela HTTPS (`https://httpbin.org/get`, apenas CONNECT):** `evidencias/atv6_https.png`

### Pergunta 6.1
> Que método HTTP aparece na sessão do `https://httpbin.org/get`? O que ele faz e por que existe?

**Resposta:** [...]

### Pergunta 6.2
> Tabela comparativa dos campos visíveis ao Fiddler em cada caso:

| Campo                          | Visível em HTTP? | Visível em HTTPS (sem decriptação)? |
|--------------------------------|------------------|-------------------------------------|
| Método                         | [...]            | [...]                               |
| URL completa (path + query)    | [...]            | [...]                               |
| Cabeçalhos de request          | [...]            | [...]                               |
| Corpo de request               | [...]            | [...]                               |
| Status code                    | [...]            | [...]                               |
| Cabeçalhos de response         | [...]            | [...]                               |
| Corpo de response              | [...]            | [...]                               |
| Host (via SNI, no `CONNECT`)   | [...]            | [...]                               |
| IP e porta de destino          | [...]            | [...]                               |

### Pergunta 6.3 (teórica)
> O que você **veria** no Fiddler se tivesse privilégio de administrador e pudesse habilitar *Decrypt HTTPS traffic*? Indique telas/abas e justifique por que essa inspeção exige a instalação de um certificado raiz.

**Resposta:** [...]

### Pergunta 6.4
> Por que a técnica de decriptação dos *debugging proxies* **não** funcionaria contra um usuário se um atacante a tentasse sem instalar o certificado?

**Resposta:** [...]

---

## Atividade 7 — Cookies e sessão (`http://httpbin.org/cookies/...`)

**Captura de tela da sequência:** `evidencias/atv7_cookies.png`

| # | URL | `Set-Cookie` recebido | `Cookie` enviado |
|---|-----|-----------------------|-------------------|
| 1 | `/cookies/set?...`       | [...] | [nenhum / ...] |
| 2 | `/cookies` (1ª visita)   | [...] | [...]          |
| 3 | `/cookies` (reload 1)    | [...] | [...]          |
| 4 | `/cookies` (reload 2)    | [...] | [...]          |

### Pergunta 7.1
> `Set-Cookie` aparece uma vez ou em toda requisição? Justifique.

**Resposta:** [...]

### Pergunta 7.2
> Que atributos o `Set-Cookie` trouxe? Explique cada um presente. Para atributos não observados, registre `não observado`.

> **Nota:** o httpbin define cookies mínimos — apenas o atributo `Path=/` estará presente. Para cada atributo ausente, registre **não observado** e explique o comportamento padrão do navegador na sua ausência (ex.: sem `Expires`/`Max-Age` → cookie de sessão; sem `Secure` → pode ser enviado por HTTP; sem `SameSite` → o navegador aplica a política padrão da versão em uso).

**Resposta:**

| Atributo  | Valor | Função | Observado? |
|-----------|-------|--------|------------|
| `Path`    | `/`   | [...]  | Sim        |
| `Domain`  | —     | [...]  | não observado |
| `Expires` | —     | [...]  | não observado |
| `Max-Age` | —     | [...]  | não observado |
| `Secure`  | —     | [...]  | não observado |
| `HttpOnly`| —     | [...]  | não observado |
| `SameSite`| —     | [...]  | não observado |

### Pergunta 7.3
> O atributo `Secure` pode aparecer num cookie recebido por HTTP puro? Qual seria o comportamento esperado? Relacione com o fato de que todo o tráfego desta atividade é visível em texto claro.

**Resposta:** [...]

### Pergunta 7.4
> Na aba **Inspectors → Cookies**, o cookie armazenado coincide com o campo `cookies` do JSON?

**Resposta:** [...]

---

## Atividade 8 — Manipulação com breakpoints

> Esta atividade usa os breakpoints interativos do Fiddler Classic.

**Captura de tela da edição do User-Agent:** `evidencias/atv8_ua_edit.png`

**JSON de resposta após edição:**

```json
{
  "user-agent": "[valor forjado]"
}
```

### Pergunta 8.1
> O servidor pode detectar que o `User-Agent` foi forjado? Discuta.

**Resposta:** [...]

### Pergunta 8.2
> Após editar a status-line de `200 OK` para `404 Not Found`, o que o navegador exibe? Comente o papel do proxy como MITM.

**Captura de tela:** `evidencias/atv8_status_edit.png`

**Resposta:** [...]

### Pergunta 8.3
> Confirme que todos os breakpoints foram desabilitados.

- [ ] Breakpoints desabilitados ao final (Shift+F11)

---

## Atividade 9 — Redirecionamento HTTP → HTTPS

**Captura de tela:** `evidencias/atv9_redir.png`

**Status-line da resposta a `http://httpbin.org/redirect-to?status_code=301&url=https%3A%2F%2Fhttpbin.org%2Fget`:**

```http
[colar aqui, ex: HTTP/1.1 301 Moved Permanently]
```

**Cabeçalho `Location` da resposta:**

```
Location: [colar aqui]
```

### Pergunta 9.1
> Código de status e cabeçalho que direcionaram o navegador para `https://`.

**Resposta:** [...]

### Pergunta 9.2
> Além do redirecionamento 3xx, qual outro mecanismo/cabeçalho faz o navegador passar a forçar HTTPS em visitas futuras? Cite a RFC.

**Resposta:** [...]

### Pergunta 9.3
> Se esse cabeçalho fosse enviado por uma resposta servida via HTTP puro, o navegador deveria obedecer? Justifique com base na RFC.

**Resposta:** [...]



### 7. Impacto prático de `Cache-Control: no-store`.

[resposta]

### 8. Como um debugging proxy decifra HTTPS sem violar a criptografia, e por que isso exige cooperação do usuário (e por que, justamente, você não pôde executar essa etapa)?

[resposta]

### 9. Exemplo de cabeçalho de request que o navegador envia automaticamente, sem a página pedir.

[resposta]

### 10. Se fosse automatizar parte da inspeção mantendo o Fiddler como proxy, que abordagem usaria? Por quê?

[resposta]

### 11. (Exclusiva do Fluxo B) Três cabeçalhos de segurança que não aparecem ou não fazem sentido em respostas HTTP puro. Para cada um, o que aconteceria se enviado por um servidor HTTP? (Cite RFC 6797 para HSTS.)

**Resposta:**

| Cabeçalho | Comportamento esperado sobre HTTP | Referência |
|-----------|-----------------------------------|-----------|
| [...]     | [...]                             | [...]     |
| [...]     | [...]                             | [...]     |
| [...]     | [...]                             | [...]     |

---

## Reflexão final (opcional, até 10 linhas)

> O que você aprendeu que não conhecia antes deste laboratório? Há algum
> cabeçalho, código de status ou comportamento que passou a olhar com
> mais atenção? Alguma dificuldade que recomendaria evitar para a próxima turma?

[reflexão]

---

## Encerramento — justificativa de segurança (Fluxo B)

**Parágrafo: por que a remoção de certificado é dispensável neste fluxo e por que seria obrigatória para o aluno administrador:**

[redigir, em até 5 linhas, com base na seção 4.6 do readme.md]

- [ ] HTTPS-First Mode / HTTPS-Only Mode reabilitado no navegador
- [ ] Fiddler fechado (porta de proxy liberada)
- [ ] Configuração de proxy removida do navegador (se aplicável)

---

## Checklist de entrega

- [ ] Todos os campos `[...]` substituídos
- [ ] Pasta `evidencias/` com capturas nomeadas por atividade (incluindo Atv. 9)
- [ ] 11 questões de verificação respondidas
- [ ] Atividade 9 (redirecionamento HTTP→HTTPS) documentada
- [ ] Justificativa de encerramento redigida
- [ ] Arquivo compactado como `NOME_RA_LAB_HTTP_FLUXOB.zip`
- [ ] Submetido no Microsoft Teams dentro do prazo
