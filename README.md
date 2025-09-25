flowchart LR
    subgraph ClientSide ["Client"]
        C(Client)
    end

    subgraph Infra ["인프라 & 배포"]
        LB(Load Balancer)

        subgraph WSCluster ["WebSocket Servers (Stateless)"]
            WS1[WebSocket Server 1]
            WS2[WebSocket Server 2]
            WS3[...]
        end
        
        subgraph MessageProcessing ["메시지 처리 시스템"]
            direction TB
            Broker[(STOMP Broker<br/>RabbitMQ)]
            Consumer[Message Consumer]
        end

        Redis[(Shared Redis<br/>Session / PubSub)]
        DB[(DB<br/>Message 저장)]
        Scheduler[Distributed Scheduler]
    end

    C -- "STOMP CONNECT" --> LB
    LB --> WS1 & WS2 & WS3

    WS1 & WS2 & WS3 -- "1. 메시지 발행(Publish)" --> Broker
    Broker -- "2. 메시지 전달" --> Consumer
    Broker -- "3. 토픽 구독한 서버로 브로드캐스트" --> WS1 & WS2 & WS3
    Consumer -- "4. 메시지 영구 저장" --> DB
    
    WS1 & WS2 & WS3 -- "세션 정보 조회/갱신" --> Redis
    Scheduler -- "예약 메시지 발행" --> Broker
    
    note right of WSCluster
      - 세션을 Redis에서 공유하므로 Stateless
      - Sticky Session 불필요
    end note
