Como tema do meu grupo foi "IoT preventiva" e o tema individual da APS2 dentro do tema principal foi "geração de OS".

No arquivo do C4 nível 2 eu não consegui encontrar uma forma de fazer as relações pararem de se sobrepor de forma a ficarem inelegíveis, infelizmente só tem como entender certos textos lendo o código devido
a essa sobreposição que ocorreram em alguns pontos do diagrama.

Diagrama de sequencia:
```mermaid
sequenceDiagram
    autonumber

    participant Sensor
    actor Operador
    participant Interface
    participant Maquina as Máquina
    participant BD as Banco de Dados
    actor Tecnico as Técnico

    loop Monitoramento contínuo
        Sensor->>Maquina: Pede por dados para telemetria
        Maquina-->>Sensor: Fornece dados

        Sensor->>Interface: Envia telemetria
        Interface->>BD: Armazena dados
        BD-->>Interface: Confirma armazenamento

        Interface->>Interface: Busca por erros

        alt Funcionamento normal
            Interface-->>Operador: Exibe status normal
        else Anomalia detectada
            Interface-->>Operador: Notifica possível falha

            Operador->>Interface: Solicita abertura de OS
            Interface->>BD: Registra Ordem de Serviço
            BD-->>Interface: Confirma criação da OS

            Interface-)Tecnico: Envia notificação de nova OS

            Tecnico->>Interface: Consulta detalhes da OS
            Interface->>BD: Busca informações da OS
            BD-->>Interface: Retorna dados da OS
            Interface-->>Tecnico: Exibe detalhes da manutenção

            Tecnico->>Maquina: Realiza manutenção preventiva
            Maquina-->>Tecnico: Retorna estado atualizado

            Tecnico->>Interface: Informa conclusão da manutenção
            Interface->>BD: Atualiza status da OS
            BD-->>Interface: Confirma atualização de status

            Interface-->>Operador: Notifica conclusão da OS
        end
    end
```

Diagrama de atividade:
```mermaid
flowchart TB

    subgraph R1["SENSOR IoT"]
        A["Coleta dados do equipamento"]
        B["Envia telemetria"]
    end

    subgraph R2["RAIA 2 - PLATAFORMA IoT"]
        D["Analisa os dados"]
        E{"Procura erro"}
        F["Não encontra falha"]
        G["Identifica possível falha"]
    end

    subgraph R3["RAIA 3 - BANCO DE DADOS"]
        H["Armazena dados"]
        I["Registra nova OS"]
        J["Atualiza status da OS"]
    end

    subgraph R4["RAIA 4 - MANUTENÇÃO"]
        K["Gera Ordem de Serviço"]
        L["Notifica técnico"]
        M["Técnico recebe a OS"]
        N["Realiza manutenção preventiva"]
        O["Registra conclusão da manutenção"]
    end

    Normal(("Status normal"))

    A --> B
    B --> D
    D --> H
    H --> E

    E -->|Não| F
    F --> Normal
    Normal --> A

    E -->|Sim| G
    G --> K
    K --> I
    I --> L
    L --> M
    M --> N
    N --> O
    O --> J
    J --> Normal
```
Diagrama de estados:
```mermaid
stateDiagram-v2
    [*] --> Monitorando

    Monitorando --> AnomaliaDetectada : TelemetriaRecebida [limitesExcedidos] / gerarAlerta()

    AnomaliaDetectada --> ValidandoAnomalia : iniciarValidacao / consultarHistorico()

    ValidandoAnomalia --> Monitorando : validacaoConcluida [falsoPositivo] / descartarAlerta()

    ValidandoAnomalia --> Registrando : validacaoConcluida [riscoConfirmado] / solicitarCriacaoOS()

    Registrando : entry / gerarNumeroOS()
    Registrando : do / validarEDepositarDados()
    Registrando : exit / confirmarRegistro()

    Registrando --> Criada : OS criada com sucesso

    Criada : entry / notificarOperador()
    Criada --> AguardandoTecnico : OS Visualizada

    AguardandoTecnico --> Atribuida : Tecnico encontrado

    Atribuida --> EmExecucao : Tecnico inicia manutenção

    EmExecucao --> Concluida : Manutenção Finalizada

    Concluida --> [*]
```
Diagrama C4 nível 2 (Visão de container):
```mermaid
C4Container
    Person(operador, "Operador", "Monitora equipamentos, analisa alertas e solicita OS")
    Person(tecnico, "Tecnico", "Recebe OS, executa manutencao e atualiza seu status")

    System_Ext(maquina, "Maquina Industrial", "Equipamento monitorado pela plataforma")
    Container_Ext(sensor, "Sensor / Gateway IoT", "ESP32 / C++", "Coleta temperatura, vibracao e pressao da maquina")

    System_Boundary(plataforma, "Plataforma IoT Preventiva") {

        Container(web, "Painel Web", "React + TypeScript", "Interface para operador e tecnico acompanharem alertas e ordens de servico")

        Container(api, "API Backend", "Node.js + NestJS", "Gerencia requisicoes, autenticacao e comunicacao entre interface e servicos")

        Container(mqtt, "Broker MQTT", "Eclipse Mosquitto 2.x", "Recebe mensagens de telemetria dos dispositivos IoT")

        Container(ingestao, "Servico de Ingestao", "Node.js + MQTT.js", "Recebe, valida e normaliza os dados enviados pelos sensores")

        ContainerQueue(rabbit, "Mensageria", "RabbitMQ 3.x", "Transporta eventos de telemetria, anomalias e ordens de servico")

        Container(analise, "Motor de Analise Preventiva", "Node.js + TypeScript", "Analisa telemetria, compara limites e identifica possiveis falhas")

        Container(os, "Servico de Ordem de Servico", "Node.js + NestJS", "Cria, atribui, acompanha e encerra ordens de servico")

        ContainerDb(banco, "Banco de Dados", "PostgreSQL 16", "Armazena usuarios, maquinas, telemetria, alertas, OS e historico de manutencao")
    }

    Rel(operador, web, "Acompanha alertas e solicita OS", "HTTPS")

    Rel(tecnico, web, "Consulta OS e atualiza manutencao", "HTTPS")

    Rel(web, api, "Envia requisicoes e consultas", "REST / JSON / HTTPS")

    Rel(api, web, "Atualiza informacoes em tempo real", "WebSocket / WSS")

    Rel(maquina, sensor, "Disponibiliza dados de funcionamento", "Modbus RTU / RS-485")

    Rel(sensor, mqtt, "Publica telemetria", "MQTT 5.0 / TLS")

    Rel(mqtt, ingestao, "Entrega mensagens dos sensores", "MQTT 5.0")

    Rel(ingestao, rabbit, "Publica telemetria.normalizada", "AMQP 0-9-1")

    Rel(rabbit, analise, "Entrega eventos de telemetria", "AMQP 0-9-1")

    Rel(analise, rabbit, "Publica anomalia.detectada", "AMQP 0-9-1")

    Rel(rabbit, os, "Entrega eventos de anomalias", "AMQP 0-9-1")

    Rel(api, os, "Solicita criacao, consulta e atualizacao de OS", "gRPC / Protocol Buffers / HTTP/2")

    Rel(os, banco, "Armazena OS, status e historico", "SQL / PostgreSQL Protocol")

    Rel(analise, banco, "Consulta limites e historico dos equipamentos", "SQL / PostgreSQL Protocol")

    Rel(os, rabbit, "Publica eventos de OS criada e atualizada", "AMQP 0-9-1")

    Rel(rabbit, api, "Entrega atualizacoes das ordens de servico", "AMQP 0-9-1")
```
