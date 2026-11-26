<div align="center">

# VexEngine-VI

**Motor e camada de execução experimental de baixo nível (DirectX 12 / x86-64)**  
Foco em estabilidade de filas, isolamento de memória CPU/GPU e *frame pacing* previsível.

[![Typing SVG](https://readme-typing-svg.demolab.com?font=Fira+Code&weight=600&size=16&pause=1200&color=38BDF8&center=true&vCenter=true&width=700&lines=DirectX+12+%2F+x86-64+Low-Level+Execution+Engine;Menus+Fluidos+%7C+Cena+Inicial:+2-5+FPS+(~14%25+VRAM+%2F+~16GB+RAM);Foco:+Isolamento+de+Copy-Back%2C+Filas+Async+e+30+%E2%86%92+60+FPS)](https://git.io/typing-svg)

<br />

![Status](https://img.shields.io/badge/Status-Em_Desenvolvimento-f59e0b?style=for-the-badge)
![Graphics](https://img.shields.io/badge/Graphics-DirectX_12-0078D4?style=for-the-badge)
![Arch](https://img.shields.io/badge/Arch-x86--64-374151?style=for-the-badge)
![GPU](https://img.shields.io/badge/Target-AMD_RDNA_2-dc2626?style=for-the-badge)
![Build](https://img.shields.io/badge/Build-0.3.0--alpha-0ea5e9?style=for-the-badge)
![Dev Window](https://img.shields.io/badge/Janela_Ativa-~3_Dias_(Pré--EXE)-22c55e?style=for-the-badge)

</div>

> **Repositório Privado:** Código 100% autoral em desenvolvimento incremental, com commits atômicos e validação contínua em hardware real. Resultados práticos já vinham sendo obtidos bem antes do vazamento do `.exe`, completando agora cerca de **3 dias** na janela de desenvolvimento/testes após a liberação dos arquivos.

### Contador de Tempo em Desenvolvimento

<p align="center">
<img src="docs/clock.svg" alt="VexEngine-VI Tempo em Desenvolvimento" width="380" />
</p>

### Navegação Rápida
- [Visão Geral](#visão-geral)
- [Matriz de Compatibilidade e Estágios](#matriz-de-compatibilidade-e-estágios)
- [Arquitetura](#arquitetura)
- [Configuração](#configuração)
- [Contador de Tempo em Desenvolvimento](#contador-de-tempo-em-desenvolvimento)
- [Roadmap](#roadmap)
- [Padrão de Commits e Notas Técnicas](#padrão-de-commits-e-notas-técnicas)

---

## Visão Geral

O **VexEngine-VI** está sendo construído de forma modular e autoral, priorizando previsibilidade na execução gráfica, isolamento de memória entre CPU e GPU e rastreabilidade completa de cada alteração.

### Pilares Técnicos
- **Execução Assíncrona de GPU:** Sincronização explícita de filas, *fences* de 64 bits e *frames in flight* para evitar bloqueios na thread de submissão e falhas de *watchdog*.
- **Isolamento de Memória e Readbacks:** Controle estrito de *copy-backs* e buffers de constantes/câmera para impedir sobrescrita entre quadros.
- **Compilação Segura de Shaders e PSOs:** Pipeline com cache persistente em armazenamento rápido e *fallback* seguro para passes pendentes.
- **Frame Pacing Desacoplado:** Separação entre o passo de simulação (`delta_time`) e a taxa de apresentação (`Present`), permitindo operar em 30 FPS ou 60 FPS sem alterar a velocidade da lógica ou da física.

---

## Matriz de Compatibilidade e Estágios

| Subsistema / Etapa | Status | Diagnóstico e Métricas Reais (RX 6650 XT / Xeon E5-2680 v4) |
| :--- | :---: | :--- |
| Memória Virtual e Alocação Base | **Funcional** | ~16 GB de RAM alocados sem vazamento crítico ou crash imediato. |
| Inicialização de GPU (D3D12 / RDNA 2) | **Funcional** | Dispositivo, filas e apresentação operacionais. |
| Tradução e Cache de Shaders | **Funcional** | Cache ativo em SSD NVMe; investigando impacto de compilação PSO em cena. |
| Mitigação de Watchdog (`0xC0000005`) | **Funcional** | Processo estável sem encerramento forçado. |
| Menus e Inicialização | **Funcional** | Navegação e apresentação fluidas. |
| Início de Jogo / Renderização de Cena | **Em Otimização** | 2–5 FPS; ~14% de VRAM (~1,1 GB) e ~16 GB de RAM (gargalo de submissão/streaming na CPU). |
| Streaming Assíncrono e Upload GPU | **Prioridade Atual** | Desbloquear transferência RAM → VRAM sem travar a thread principal. |
| Modo Desbloqueado de 60 FPS | **Planejado** | Será validado após estabilizar o fluxo de cena em 30 FPS. |

---

## Arquitetura

O **VexEngine-VI** separa responsabilidades críticas para evitar bloqueios na thread principal de renderização e facilitar reversões rápidas em caso de regressão:

```text
VexEngine-VI/
├── core/        # Memória virtual, timers de alta precisão e controle de watchdog
├── gpu/         # Backend gráfico, filas assíncronas, fences 64-bit e frame pacing
├── shader/      # Tradução de shaders, cache persistente de PSOs e fallback seguro
├── io/          # Streaming assíncrono de assets, controle de readback e VRAM
├── cli/         # Ponto de entrada e preparação para integração com VexLauncher
├── config/      # Arquivos de configuração (ex.: target_fps, caminhos de cache)
└── README.md    # Documentação técnica e registro de evolução
