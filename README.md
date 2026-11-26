# VexEngine-VI

VexEngine-VI Ã© um projeto autoral de engenharia de execuÃ§Ã£o e renderizaÃ§Ã£o para PC, desenvolvido em etapas com prioridade para correÃ§Ã£o, previsibilidade e estabilidade.

O projeto estÃ¡ em fase inicial. Ainda nÃ£o Ã© um port funcional, nÃ£o executa GTA VI e nÃ£o oferece compatibilidade grÃ¡fica completa. O objetivo atual Ã© construir e validar uma base prÃ³pria de temporizaÃ§Ã£o, propriedade por quadro, memÃ³ria, comunicaÃ§Ã£o D3D12 e apresentaÃ§Ã£o DXGI.

## Tempo de desenvolvimento

<img src="docs/clock.svg" alt="VexEngine-VI Tempo em Desenvolvimento" width="560" />

O contador parte de 23 nov. 2026, 11:36 BRT (UTCâˆ’03:00) e o SVG Ã© atualizado automaticamente a cada cinco minutos pelo GitHub Actions. Para acompanhar os segundos ao vivo localmente, abra [vex-clock.html](vex-clock.html) no navegador.

## PrincÃ­pios

- ImplementaÃ§Ã£o prÃ³pria; projetos externos servem apenas como referÃªncias tÃ©cnicas e fontes de hipÃ³teses.
- Nenhuma espera ativa ou bloqueio sÃ­ncrono desnecessÃ¡rio na thread de submissÃ£o.
- Valores de fence de 64 bits monotÃ´nicos e verificÃ¡veis.
- Recursos de um quadro nÃ£o podem ser reutilizados antes da conclusÃ£o de todas as filas que os utilizam.
- Dados de cÃ¢mera sÃ£o capturados em snapshots imutÃ¡veis por quadro.
- SimulaÃ§Ã£o e frequÃªncia de apresentaÃ§Ã£o devem ser desacopladas.
- OrÃ§amento de memÃ³ria explÃ­cito, preservando margem para sistema e driver.
- CÃ³digo inseguro fica restrito aos mÃ³dulos de integraÃ§Ã£o com APIs nativas.

## Estado atual

| Subsistema | Estado |
|---|---|
| Workspace Rust | Inicializado |
| Acumulador de simulaÃ§Ã£o fixa | Implementado + testes |
| Slots de quadros / timelines | PolÃ­tica CPU implementada + testes |
| Snapshot imutÃ¡vel de cÃ¢mera | Implementado + testes |
| OrÃ§amento contÃ¡bil de memÃ³ria | Implementado + testes |
| EnumeraÃ§Ã£o DXGI e seleÃ§Ã£o de adaptador | Testada na RX 6650 XT |
| CriaÃ§Ã£o de dispositivo D3D12 | Testada na RX 6650 XT |
| Fila direta e fence de 64 bits | Testadas no hardware |
| TrÃªs pares de command allocator/list | Seis submissÃµes reais concluÃ­das |
| Janela Win32 e swap chain DXGI | Criada e executada na RX 6650 XT |
| Render target, clear e transiÃ§Ãµes | Executados sem erro reportado pelo D3D12 |
| Present e sincronizaÃ§Ã£o por back buffer | Executados; fence final concluÃ­da; confirmaÃ§Ã£o visual humana pendente |
| Resize seguro da swap chain | Implementado e testado com 3 redimensionamentos externos (4 eventos WM_SIZE) |
| DiagnÃ³stico de device removed | Parcial: HRESULT de Present/ResizeBuffers + GetDeviceRemovedReason; recuperaÃ§Ã£o pendente |
| Fontes HLSL + bytecode VS/PS 5.1 | Compilados; pipeline ainda nÃ£o conectado |
| Geometria / draw calls | NÃ£o implementados |
| Upload heaps / streaming | NÃ£o implementados |
| IntegraÃ§Ã£o com GTA VI | NÃ£o implementada |
| Contrato tipado de configuraÃ§Ãµes do runtime | Implementado + 5 testes |
| ClassificaÃ§Ã£o Live / RestartGame | Implementada; ainda sem painel/IPC |
| Meta de 60 FPS | Planejada, nÃ£o habilitada |

## Compilar e testar

Requer Rust stable e Windows SDK.

Testes do nÃºcleo:

    cargo test --workspace
    cargo clippy --workspace --all-targets --features windows-backend -- -D warnings

Probe de comunicaÃ§Ã£o D3D12:

    cargo run -p vex-core --features windows-backend --bin vex-gpu-probe

Primeiro teste visual DXGI (janela 1280x720; 15 segundos por padrÃ£o, duraÃ§Ã£o configurÃ¡vel):

    cargo run -p vex-core --features windows-backend --bin vex-visual-test
    cargo run -p vex-core --features windows-backend --bin vex-visual-test -- 60

O teste visual foi executado por 60 segundos na RX 6650 XT: 3.547 apresentaÃ§Ãµes foram submetidas e a fence final foi concluÃ­da em 3.547. A janela Win32 foi capturada visÃ­vel com o render target preenchido; essa validaÃ§Ã£o cobre apenas clear + Present, nÃ£o geometria, shaders ou desempenho do jogo. A pausa de 16 ms do utilitÃ¡rio Ã© proteÃ§Ã£o contra saturaÃ§Ã£o da CPU e nÃ£o constitui benchmark nem comprova 60 FPS.

## Estrutura

    crates/vex-core/src/frame/          Posse de slots e snapshots
    crates/vex-core/src/timing/         TemporizaÃ§Ã£o de simulaÃ§Ã£o
    crates/vex-core/src/memory/         OrÃ§amento de memÃ³ria
    crates/vex-core/src/runtime_settings.rs
                                        Contrato e validaÃ§Ã£o de configuraÃ§Ãµes
    crates/vex-core/src/platform/d3d12.rs
                                        Dispositivo, fila e fences D3D12
    crates/vex-core/src/platform/presentation.rs
                                        Janela, swap chain e apresentaÃ§Ã£o DXGI
    crates/vex-core/src/bin/            Ferramentas de validaÃ§Ã£o
    assets/shaders/                     Fontes HLSL e bytecode compilado
    docs/architecture/                  Arquitetura e fluxos tÃ©cnicos
    docs/decisions/                     DecisÃµes de projeto
    docs/TEST-PLAN.md                   Plano de testes

## Hardware de referÃªncia

- AMD Radeon RX 6650 XT â€” 8 GB de VRAM
- Intel Xeon E5-2680 v4
- 32 GB de RAM
- Windows 10 Pro 22H2

O probe identificou a RX 6650 XT (Vendor ID 0x1002), com 7,96 GiB de memÃ³ria dedicada reportada, criou o dispositivo D3D12, submeteu seis command lists e confirmou o progresso da fence atÃ© o valor 7.

## Roadmap

1. Confirmar apresentaÃ§Ã£o DXGI estÃ¡vel e encerramento limpo.
2. Implementar resize e tratamento de perda de dispositivo.
3. Adicionar geometria de teste e shaders compilados.
4. Criar buffers de upload e recursos imutÃ¡veis por quadro.
5. Instrumentar frame pacing, latÃªncia e orÃ§amento dinÃ¢mico de memÃ³ria.
6. Implementar streaming e cache de pipelines/shaders.
7. Desacoplar simulaÃ§Ã£o e apresentaÃ§Ã£o; avaliar 60 FPS.
8. Planejar integraÃ§Ã£o progressiva com o ambiente-alvo.

## DocumentaÃ§Ã£o

- [FundaÃ§Ã£o tÃ©cnica](docs/architecture/FOUNDATION.md)
- [Probe D3D12](docs/architecture/WINDOWS-D3D12-PROBE.md)
- [ApresentaÃ§Ã£o DXGI](docs/architecture/DXGI-PRESENTATION.md)
- [Plano de testes](docs/TEST-PLAN.md)
- [Painel e configuraÃ§Ãµes do runtime](docs/architecture/RUNTIME-CONTROL-PLANE.md)
- [Primeiros shaders](docs/architecture/FIRST-SHADERS.md)
- [Changelog](CHANGELOG.md)

## Integridade

O probe D3D12 e o caminho inicial de apresentaÃ§Ã£o foram executados no hardware de referÃªncia. A fila apresentou 832 quadros de clear e concluiu a fence correspondente. A aprovaÃ§Ã£o visual depende de confirmaÃ§Ã£o na tela; ainda nÃ£o alegamos renderizaÃ§Ã£o de geometria, estabilidade prolongada, desempenho em jogo ou 60 FPS. A licenÃ§a existente no repositÃ³rio Ã© GNU GPL versÃ£o 2 e permanece inalterada; a decisÃ£o de licenciamento deve ser revisada antes de qualquer distribuiÃ§Ã£o pÃºblica.

