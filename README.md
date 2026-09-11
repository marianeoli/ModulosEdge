# SafeLiving — Edge, Fog & Cloud

Simulação de uma rede de casas inteligentes ("SafeLiving") organizada nas três camadas de computação distribuída: **Edge** (dentro de cada casa), **Fog** (antena OpenRAN do bairro) e **Cloud** (nuvem central).

Como trabalho da disciplina de Redes Sem Fio, este repositório contém a parte do grupo responsável pela **Edge**: os dispositivos de processamento e abstração dentro de cada residência.

## Arquitetura

**Edge** | Gateway dentro de cada casa | `DispositivoResidencial` (sensores) + `CasaInteligente` (gateway) |
**Fog** | Antena OpenRAN do bairro | `EstacaoORAN` |
**Cloud** | Servidor central | `NucleoCentral` |

Cada camada segue os mesmos três princípios:
- **Transmissão** — como o dado sobe (latência simulada até a próxima camada)
- **Processamento** — que decisão é tomada, e em que camada ela é tomada
- **Abstração** — o que é enviado adiante (nunca o dado bruto, só o essencial)

A ideia central: decisões urgentes e locais acontecem **na Edge, em milissegundos**, sem depender de rede externa, só o resultado já processado sobe para a Fog e a Cloud.

## Módulos da Edge implementados neste trabalho

Além da regra original do estudo de caso (fusão câmera + sensor de presença para liberar o portão), o grupo adicionou dois novos módulos de sensor/processamento/abstração ao gateway residencial:

### 1. Sensor de fumaça/gás

Simula um sensor de cozinha que raramente (~5% por ciclo) detecta uma concentração de gás acima do limite de segurança (`LIMIAR_GAS_PPM = 300`).

- **Processamento:** o gateway decide sozinho, sem esperar Fog/Cloud. E então, corta a válvula de gás e aciona a sirene.
- **Abstração:** sobe para a Fog `{casa, nível medido, urgente=True, hora}`. O nível fica disponível porque pode ser útil para a Fog correlacionar emergências entre casas vizinhas (fora do escopo deste módulo).

### 2. Sensor de queda do idoso

Módulo de saúde domiciliar com monitoramento não invasivo. Simula raramente (~6% por ciclo) uma queda detectada.

- **Processamento:** o gateway reage localmente e imediatamente. E então destranca a porta principal, acende todas as luzes e desliga o fogão, para facilitar a entrada de socorristas.
- **Abstração:** sobe para a Fog só o essencial: `{casa, hora, prioridade="CRITICA", necessita_ambulancia=True}`. Nunca dados médicos ou de vídeo brutos.

Os dois módulos foram desenhados para não interferir na regra original de acesso por veículo e podem disparar juntos no mesmo ciclo, cada um com seu próprio evento.

## Como rodar

Requer apenas Python 3 (sem dependências externas).

```bash
python3 safeliving_edge_fog_cloud.py
```

A simulação roda `CICLOS_DE_SIMULACAO = 8` ciclos, imprime no terminal o que acontece em cada camada, gera um dashboard (`dashboard_safeliving.html`) e, ao final, abre um menu para explorar os dados coletados pela Cloud.

Como os eventos de fumaça/gás e queda são raros e aleatórios, é normal que nem sempre apareçam em uma única execução. Nesse caso, rode mais de uma vez se quiser vê-los.

## Estrutura do projeto

```
safeliving_edge_fog_cloud.py   # simulação completa (Edge, Fog, Cloud)
dashboard_safeliving.html      # dashboard gerado pela Cloud
estudo_de_caso_edge_fog_cloud.pdf  # estudo de caso original da disciplina
```

## O que fica em aberto para a Fog

Hoje, os eventos `EMERGENCIA_GAS_DETECTADO` e `EMERGENCIA_MEDICA_QUEDA` chegam ao buffer da `EstacaoORAN`, mas não têm regra própria em `xapp_processar_lote()`. Ou seja, são recebidos mas não geram nenhuma ação na Fog nem chegam à Cloud. Isso é esperado: a integração entre esses alertas e a camada Fog (por exemplo, correlacionar emergências entre casas vizinhas) não é responsabilidade deste módulo.

## Contribuidores do trabalho

- Mariane
- João Marcos
- Carlos Daniel 
