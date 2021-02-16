---
title: "[SSE] Node와 Vue에서 SSE Connection 설정하기 🕊️"
date: 2021-02-15 00:00:00 -0400
categories: sse javascript nodejs vue
---

> Node기반의 DB batch update의 실시간 업데이트 정보를 vue 기반의 front에서 보여주고 싶었다. 처음에는 socket 커넥션을 생각했으나, node -> vue로의 단방향 데이터 전달만 필요하므로 SSE연결로 진행하였다.

websocket 프로토콜은 클라이언트와 서버 사이에 양방향 커넥션을 제공한다. 이런 양방향 커넥션은 채팅이나 멀티플레이어 게임등에서 유용하다. 하지만, 서버에서 클라이언트로 데이터를 보내주기만 하면 되는 경우도 있다. (뉴스피드, 주식 가격, 게임 스코어 등등.. ) 이럴 때는 SSE(Server-Sent Event)를 사용하는것이 적합하다.

# SSE (Server Sent Event)란?
* 서버에서 연결된 클라이언트로 **단방향**으로 이벤트를 보내는 방식
* 서버에서 푸시 알람을 보내는 등에 주로 이용
* HTTP 프로토콜 기반 (content type: Text/Event-Stream)
* 한 번 맺어둔 커넥션을 통해 로 지속적으로 데이터를 전송함
* 클라이언트에서 연결을 해제하거나 웹페이지를 나갈 때 커넥션이 종료됨

<img src="/assets/2021-02-15-node_sse_connection/sse.png" alt="what is sse?"/>
[이미지출처](https://www.pubnub.com/learn/glossary/what-are-server-sent-events)

# 서버(Node)쪽 설정
### set connection
클라이언트에서 /sse로 요청을 보내면 SSE 커넥션을 맺은 후 해당 커넥션 정보를 저장한다
```javascript
let { sseConnections} = require('../tasks/sseHandler')

...

/* GET home page. */
router.get('/sse', async function (req, res, next) {
    const headers = {
        'Content-Type': 'text/event-stream',
        'Connection': 'keep-alive',
        'Cache-Control': 'no-cache'
    };
    res.writeHead(200, headers);
    const clientId = Date.now();
    sseConnections[clientId] = res // 커넥션 저장
    console.log(`${clientId} Connection established`);

    req.on('close', () => {
        console.log(`${clientId} Connection closed`);
        delete sseConnections[clientId]
    });
});
```

*  커넥션을 저장해두었다가 이벤트 보낼 때 다시 꺼내서 사용해야 함
 => sseConnections라는 빈 객체를 만들어두고, 그쪽에 커넥션들을 저장해둠
* 커넥션이 종료되면 해당 커넥션에 더이상 이벤트를 보내지 말아야 함
=> delete sseConnections[clientId]



### Send SSE Event
연결된 connection들에 이벤트를 전송한다
``` javascript
/* sseHandler.js */
const sseConnections = {};

exports.sendPing = (message) => {
    Object.values(sseConnections).forEach(c => {
        c.write(`event: ping\n`)
        c.write(`data: ${message}\n\n`)
    })
};

exports.sendMessage = (target, status, message) => {
    const data = { target, status, message }
    Object.values(sseConnections).forEach(c => {
        c.write(`data: ${JSON.stringify(data)}\n\n`)
    })
};
exports.sseConnections = sseConnections;

```
* message type 이벤트
 => 그냥 connection에 `write("data: [보내려는 메시지]\n\n")` 형태로 전송
* 이벤트 타입 설정
=> `write("event: [이벤트이름]\n data: [보내려는 메시지]\n\n")`
(이렇게 그냥 텍스트로 이벤트 이름을 이어붙이면 되는거였음...)


필요할때 sendPing, sendMessage 함수를 호출해 사용하면 된다
```javascript
const { sendMessage } = require('../tasks/sseHandler')
...
sendMessage({db: table, operation: 'READ' }, STATUS.READY, `reading database...`)
```

# 클라이언트(Vue)설정
### EventSource
* SSE에 대한 웹 콘텐츠 인터페이스
* text/event-stream 포맷으로 이벤트를 보내는 HTTP 서버에 지속적인 연결을 맺고, 들어오는 이벤트 처리할 수 있음
* 각각 이벤트에 맞게 적절한 리스너를 연결하여 처리하면 됨
참고 : [document](https://developer.mozilla.org/ko/docs/Web/API/EventSource)

### setSSEConnection
필요한 부분에서 setSSEConnection를 호출하여 사용한다.
```javascript

@Component
export default class ExampleComponent extends Vue {

  mounted () {
    //사용자가 페이지를 떠나기 전에 발생하는 이벤트
    window.onbeforeunload = () => {
        // 페이지를 닫을 때 SSE 커넥션 닫기
        this.es && this.es.close()
    }
  }

  // 버튼 누르면 SSE connection 설정!
  async onClickButton() {
      this.setSSEConnection()
      await this.sleep(1000) // 첫번째 이벤트가 너무 빨리 도착해서 프론트에서 이벤트 인식 못하는 것 방지
  }

  sleep(ms) { return new Promise(resolve => setTimeout(resolve, ms)); }

  listener (event) {
      console.log(`${event.type}: ${event.data || this.es.url}`);
  };

  triggerListener(event){
    // finished 라는 데이터로 ping 이벤트가 들어오면 connection을 종료함.
    if(event.data === 'finished'){
        this.es.close()
        // connection이 닫히면 EventSource의 readyState프로퍼티 값이 2가 된다.
        // 이를 이용해 커넥션이 정상적으로 종료되었는지 확인한다.
        console.log(this.es.readyState=== 2 ? 'connection closed' : 'connection not closed')

    }
  }

  setSSEConnection(){
      this.es = new EventSource(`${url}`);
      this.es.addEventListener('open', this.listener);
      this.es.addEventListener('ping', this.pingListener);
      this.es.addEventListener('message', this.msgListener);
      this.es.addEventListener('error', this.listener);
  }
}
```

필요한 부분에서 커넥션을 맺고 끊으면 되는데, 이 프로젝트에서는 **사용자가 버튼을 클릭하면 연결을 맺었고, 서버쪽에서 종료 메시지가 도착하면 연결을 끊었다.**
