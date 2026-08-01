# Setup do Mobile

Guia completo para configurar e executar o aplicativo mobile do TepConfina.

## Pre-requisitos

| Ferramenta      | Versao Minima | Plataforma         |
|-----------------|---------------|--------------------|
| Flutter SDK     | 3.x           | Todas              |
| Dart            | 3.x           | Incluso no Flutter |
| Android Studio  | 2024.x        | Android            |
| Xcode           | 15+           | iOS (somente macOS)|
| Java JDK        | 17            | Android            |
| CocoaPods       | 1.14+         | iOS (somente macOS)|

!!! warning "Xcode somente macOS"
    O desenvolvimento para iOS requer macOS com Xcode instalado. Para desenvolvimento exclusivo Android, Linux e Windows tambem sao suportados.

## Clonando o Repositorio

```bash
git clone git@github.com:TecnoePec/tepconfina-mobile.git
cd tepconfina-mobile
```

## Verificando o Ambiente

Execute o diagnostico do Flutter:

```bash
flutter doctor -v
```

Todos os itens devem estar marcados com checkmark verde. Resolva quaisquer pendencias antes de prosseguir.

## Instalando Dependencias

```bash
flutter pub get
```

### Principais Dependencias

| Pacote              | Funcao                            |
|---------------------|-----------------------------------|
| flutter_riverpod    | Gerenciamento de estado           |
| hive / hive_flutter | Armazenamento local               |
| dio                 | Cliente HTTP                      |
| go_router           | Navegacao                         |
| freezed             | Imutabilidade e union types       |
| mocktail             | Mocks para testes                |

## Configuracao de Ambiente

A URL da API **não** vem de um `.env` — é resolvida por `--dart-define=ENV` em `lib/core/utils/constants.dart`:

| `ENV`        | baseUrl                                             |
|--------------|-----------------------------------------------------|
| (default/dev)| `http://10.0.2.2:5000/api` (Android) / `http://localhost:5000/api` (iOS) |
| `production` | `https://tepconfina-api.tecnoepec.com.br/api`       |

```bash
flutter run                              # dev (localhost)
flutter run --dart-define=ENV=production # produção
```

!!! info "IP do emulador Android"
    O endereco `10.0.2.2` e o alias do `localhost` da maquina host no emulador Android. Para dispositivos fisicos, use o IP da rede local.

## Configuracao do Firebase (obrigatoria)

O app **depende do Firebase para iniciar**. Os arquivos de config sao versionados no repo (projeto `tep-confina`):

- Android: `android/app/google-services.json`
- iOS: `ios/Runner/GoogleService-Info.plist`

Se precisar regenerar, registre o app no [Firebase console](https://console.firebase.google.com) com o bundle `br.com.tecnoepec.tepconfina`, ou rode `flutterfire configure --project=tep-confina`.

## Executando o Projeto

### Android

```bash
# Listar dispositivos disponiveis
flutter devices

# Executar no emulador ou dispositivo conectado
flutter run
```

### iOS (somente macOS)

Plataforma iOS adicionada em 2026-07-29. `GoogleService-Info.plist` já vem no repo.

```bash
# Instalar dependencias nativas (primeira vez / ao mudar deps nativas)
cd ios && pod install && cd ..

# Executar num simulador (ex.: iPhone 16)
flutter run -d "iPhone 16" --dart-define=ENV=production
```

!!! warning "Simulador iOS: só debug"
    O simulador iOS suporta apenas modo **debug** — `--release`/`--profile` só funcionam em device fisico (que exige conta Apple Developer + assinatura).

### Modo de desenvolvimento

```bash
# Hot reload automatico
flutter run --debug
```

## Estrutura do Projeto

```
lib/
  core/
    config/         # Configuracoes da app
    services/       # AuthService, ApiService, StorageService
    theme/          # Tema e cores
  features/
    auth/           # Login, registro
    dashboard/      # Tela principal
    lotes/          # Gestao de lotes
    animais/        # Gestao de animais
    pesagens/       # Registro de pesagens
  shared/
    models/         # Modelos de dados
    providers/      # Riverpod providers
    widgets/        # Widgets reutilizaveis
```

## Comandos Uteis

| Comando                          | Descricao                        |
|----------------------------------|----------------------------------|
| `flutter pub get`                | Instala dependencias             |
| `flutter run`                    | Executa o app                    |
| `flutter test`                   | Roda os testes                   |
| `flutter build apk`             | Gera APK de release              |
| `flutter build ios`             | Gera build iOS                   |
| `flutter clean`                  | Limpa cache e build              |
| `dart run build_runner build`    | Gera codigo (freezed, json)      |

## Geracao de Codigo

Apos alterar modelos com anotacoes `@freezed` ou `@JsonSerializable`:

```bash
dart run build_runner build --delete-conflicting-outputs
```

!!! tip "Watch mode"
    Use `dart run build_runner watch` durante o desenvolvimento para regenerar codigo automaticamente.

!!! success "Verificacao"
    O app deve abrir na tela de login. Utilize as mesmas credenciais do ambiente web para autenticar.
