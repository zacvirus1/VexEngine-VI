<div align="center">

# VexEngine-VI

**Motor e camada de execução experimental de baixo nível (DirectX 12 / x86-64)**  
Foco em estabilidade de filas, isolamento de memória CPU/GPU e *frame pacing* previsível.

[![Typing SVG](https://readme-typing-svg.demolab.com?font=Fira+Code&size=22&pause=1000&color=10B981&center=true&vCenter=true&width=700&lines=VexEngine-VI;DirectX+12+%7C+x86-64;Engine+Experimental+de+Baixo+N%C3%ADvel)](https://github.com/zacvirus1/VexEngine-VI)

<br />

![Status](https://img.shields.io/badge/status-em%20desenvolvimento-10b981?style=for-the-badge)
![Graphics](https://img.shields.io/badge/graphics-DirectX%2012-0078D4?style=for-the-badge&logo=microsoft)
![Arch](https://img.shields.io/badge/arch-x86--64-176f5d?style=for-the-badge)
![GPU](https://img.shields.io/badge/GPU-RX%206650%20XT-ED1C24?style=for-the-badge&logo=amd)
![Build](https://img.shields.io/badge/build-experimental-f3bd4f?style=for-the-badge)
![Platform](https://img.shields.io/badge/platform-Windows-2F81F7?style=for-the-badge&logo=windows)

</div>

> **Repositório Privado:** Código 100% autoral em desenvolvimento incremental, com commits atômicos e validação contínua em hardware real. Resultados práticos já vinham sendo obtidos bem antes da publicação do executável.

### Contador de Tempo em Desenvolvimento

<p align="center">
  <img src="https://raw.githubusercontent.com/zacvirus1/VexEngine-VI/main/docs/clock.svg" alt="VexEngine-VI Tempo em Desenvolvimento" width="380" />
</p>

### Navegação Rápida
- [Visão Geral](#visão-geral)
- [Matriz de Compatibilidade e Estágios](#matriz-de-compatibilidade-e-estágios)
- [Arquitetura](#arquitetura)
- [Contador de Tempo em Desenvolvimento](#contador-de-tempo-em-desenvolvimento)

---

## Visão Geral

O **VexEngine-VI** está sendo construído de forma modular e autoral, priorizando previsibilidade na execução gráfica, isolamento de memória entre CPU e GPU e rastreabilidade completa de cada subsistema.

### Pilares Técnicos
- **Execução Assíncrona de GPU:** Sincronização explícita de filas, *fences* de 64 bits e *frames in flight* para evitar bloqueios na thread de submissão e falhas de *watchdog*.
- **Isolamento de Memória e Readbacks:** Controle estrito de *copy-backs* e buffers de constantes/câmera para impedir sobrescrita entre quadros.
- **Compilação Segura de Shaders e PSOs:** Pipeline com cache persistente em armazenamento rápido e *fallback* seguro para passes pendentes.
- **Frame Pacing Desacoplado:** Separação entre o passo de simulação (`delta_time`) e a taxa de apresentação (`Present`), permitindo operar em 30 FPS ou 60 FPS sem alterar a velocidade da lógica.

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
```
