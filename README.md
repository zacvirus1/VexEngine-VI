# VexEngine-VI

Motor e camada de execução experimental de baixo nível focado em estabilidade, baixa latência, gerenciamento assíncrono de GPU e controle preciso de *frame pacing*.

> **Status do Projeto:** Em desenvolvimento ativo (repositório privado para versionamento incremental e auditoria de estabilidade).

---

## Visão Geral

O **VexEngine-VI** está sendo construído de forma modular e autoral, priorizando previsibilidade na execução gráfica, isolamento de memória entre CPU e GPU e rastreabilidade completa de cada alteração via commits atômicos.

### Pilares Técnicos
- **Execução Assíncrona de GPU:** Sincronização explícita de filas, *fences* de 64 bits e *frames in flight* para evitar bloqueios na thread de submissão e falhas de *watchdog*.
- **Isolamento de Memória e Readbacks:** Controle estrito de *copy-backs* e buffers de constantes/câmera para impedir sobrescrita entre quadros.
- **Compilação Segura de Shaders e PSOs:** Pipeline com cache persistente em armazenamento rápido e *fallback* seguro para passes pendentes.
- **Frame Pacing Desacoplado:** Separação entre o passo de simulação (`delta_time`) e a taxa de apresentação (`Present`), permitindo operar em 30 FPS ou 60 FPS sem alterar a velocidade da lógica ou da física.

---

## Estado Atual do Desenvolvimento

| Componente / Etapa | Status | Observações |
| :--- | :--- | :--- |
| Inicialização de GPU (D3D12) | Estável | Dispositivo, filas e alocação base operacionais. |
| Pipeline Básico de Shaders | Estável | Carregamento inicial validado. |
| Tratamento de Watchdog (`0xC0000005`) | Em ajuste | Bypass inicial validado; foco atual na causa raiz de sincronização de filas. |
| Renderização de Abertura | Em teste (Build 3) | Tela preta superada; imagem exibida, ajustando estabilidade de câmera/profundidade entre quadros. |
| Desbloqueio Funcional de 60 FPS | Planejado | Será ativado após estabilização completa da imagem em 30 FPS. |
| Integração com **VexLauncher** | Planejado | Estrutura preparada para futura interface dedicada de inicialização e configuração. |

---

## Estrutura Planejada do Repositório

```text
VexEngine-VI/
├── core/        # Gerenciamento de memória, timers de alta precisão e threading
├── gpu/         # Backend gráfico, filas assíncronas, fences e frame pacing
├── shader/      # Tradução, cache persistente e compilação segura de PSOs
├── io/          # Streaming assíncrono de assets e controle de readback
├── cli/         # Ponto de entrada e preparação para integração com VexLauncher
├── config/      # Arquivos de configuração (ex.: target_fps, caminhos de cache)
└── README.md    # Documentação técnica e registro de evolução
