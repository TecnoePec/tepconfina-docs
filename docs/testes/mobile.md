# Testes do Mobile

Estrategia e estrutura de testes do aplicativo Flutter do TepConfina.

## Framework e Ferramentas

| Ferramenta      | Versao | Funcao                              |
|-----------------|--------|-------------------------------------|
| flutter_test    | -      | Framework de testes nativo Flutter  |
| mocktail        | 1.0.4  | Biblioteca de mocking               |

## Estrutura do Projeto

```
test/
  core/
    services/
      auth_service_test.dart
      api_service_test.dart
      storage_service_test.dart
    config/
      environment_test.dart
  features/
    auth/
      login_screen_test.dart
      auth_provider_test.dart
    dashboard/
      dashboard_screen_test.dart
      dashboard_provider_test.dart
    lotes/
      lotes_list_test.dart
      lote_detail_test.dart
      lote_form_test.dart
      lote_provider_test.dart
    animais/
      animal_list_test.dart
      animal_provider_test.dart
    pesagens/
      pesagem_form_test.dart
      pesagem_provider_test.dart
  shared/
    models/
      lote_model_test.dart
      animal_model_test.dart
      pesagem_model_test.dart
      user_model_test.dart
    providers/
      theme_provider_test.dart
    widgets/
      custom_text_field_test.dart
      loading_indicator_test.dart
  helpers/
    pump_app.dart
    mock_providers.dart
```

## Total de Testes: 140

### Distribuicao por Categoria

| Categoria         | Qtd  | Descricao                              |
|-------------------|------|----------------------------------------|
| Core Services     | 25   | AuthService, ApiService, StorageService|
| Providers         | 35   | Riverpod providers e notifiers         |
| Models            | 30   | Serializacao, validacao, metodos       |
| Features (UI)     | 30   | Testes de widgets por feature          |
| Shared Widgets    | 20   | Componentes reutilizaveis              |

## Padroes de Mock

### MockDio

```dart
class MockDio extends Mock implements Dio {}

void main() {
  late MockDio mockDio;
  late ApiService apiService;

  setUp(() {
    mockDio = MockDio();
    apiService = ApiService(dio: mockDio);
  });

  test('deve buscar lotes com sucesso', () async {
    when(() => mockDio.get('/api/lotes')).thenAnswer(
      (_) async => Response(
        data: [
          {'id': '1', 'nome': 'Lote 001', 'status': 'Ativo'},
        ],
        statusCode: 200,
        requestOptions: RequestOptions(path: '/api/lotes'),
      ),
    );

    final lotes = await apiService.getLotes();
    expect(lotes, isNotEmpty);
    expect(lotes.first.nome, equals('Lote 001'));
  });
}
```

### MockHiveBox

```dart
class MockHiveBox extends Mock implements Box<dynamic> {}

void main() {
  late MockHiveBox mockBox;
  late StorageService storageService;

  setUp(() {
    mockBox = MockHiveBox();
    storageService = StorageService(box: mockBox);
  });

  test('deve salvar token no Hive', () async {
    when(() => mockBox.put('token', any())).thenAnswer((_) async {});

    await storageService.saveToken('jwt-token-aqui');

    verify(() => mockBox.put('token', 'jwt-token-aqui')).called(1);
  });

  test('deve recuperar token do Hive', () {
    when(() => mockBox.get('token')).thenReturn('jwt-token-aqui');

    final token = storageService.getToken();
    expect(token, equals('jwt-token-aqui'));
  });
}
```

### MockAuthService

```dart
class MockAuthService extends Mock implements AuthService {}

void main() {
  late MockAuthService mockAuthService;

  setUp(() {
    mockAuthService = MockAuthService();
  });

  test('deve autenticar com credenciais validas', () async {
    when(() => mockAuthService.login('admin@tepconfina.com', 'Admin@123'))
        .thenAnswer((_) async => AuthResult(
              token: 'jwt-token',
              refreshToken: 'refresh-token',
              user: User(nome: 'Admin', email: 'admin@tepconfina.com'),
            ));

    final result = await mockAuthService.login(
        'admin@tepconfina.com', 'Admin@123');

    expect(result.token, isNotEmpty);
    expect(result.user.nome, equals('Admin'));
  });
}
```

## Widget Tests

### Helper pumpApp

```dart
// helpers/pump_app.dart
extension PumpApp on WidgetTester {
  Future<void> pumpApp(Widget widget, {List<Override>? overrides}) async {
    await pumpWidget(
      ProviderScope(
        overrides: overrides ?? [],
        child: MaterialApp(
          home: widget,
          theme: TepConfinaTheme.light,
        ),
      ),
    );
    await pump();
  }
}
```

### Exemplo de Widget Test

```dart
void main() {
  testWidgets('LoginScreen deve exibir campos de email e senha',
      (tester) async {
    await tester.pumpApp(const LoginScreen());

    expect(find.byType(TextField), findsNWidgets(2));
    expect(find.text('Email'), findsOneWidget);
    expect(find.text('Senha'), findsOneWidget);
    expect(find.text('Entrar'), findsOneWidget);
  });

  testWidgets('LoginScreen deve mostrar erro com credenciais invalidas',
      (tester) async {
    final mockAuthService = MockAuthService();
    when(() => mockAuthService.login(any(), any()))
        .thenThrow(AuthException('Credenciais invalidas'));

    await tester.pumpApp(
      const LoginScreen(),
      overrides: [authServiceProvider.overrideWithValue(mockAuthService)],
    );

    await tester.enterText(find.byKey(Key('email')), 'wrong@email.com');
    await tester.enterText(find.byKey(Key('password')), 'wrongpass');
    await tester.tap(find.text('Entrar'));
    await tester.pump();

    expect(find.text('Credenciais invalidas'), findsOneWidget);
  });
}
```

## Testes de Modelos

```dart
void main() {
  group('LoteModel', () {
    test('deve criar a partir de JSON', () {
      final json = {
        'id': '123',
        'nome': 'Lote 001',
        'quantidadeAnimais': 30,
        'pesoLiquidoEntrada': 15000.0,
        'status': 'Ativo',
      };

      final lote = LoteModel.fromJson(json);
      expect(lote.nome, equals('Lote 001'));
      expect(lote.pesoMedioEntrada, equals(500.0));
    });

    test('deve serializar para JSON', () {
      final lote = LoteModel(nome: 'Lote 001', quantidadeAnimais: 30);
      final json = lote.toJson();
      expect(json['nome'], equals('Lote 001'));
    });
  });
}
```

## Executando os Testes

| Comando                          | Descricao                        |
|----------------------------------|----------------------------------|
| `flutter test`                   | Executar todos os testes         |
| `flutter test test/core/`        | Executar testes de uma pasta     |
| `flutter test --coverage`        | Gerar relatorio de cobertura     |
| `flutter test --reporter expanded` | Output detalhado              |

!!! tip "Cobertura"
    Apos gerar cobertura com `--coverage`, use `genhtml` para criar um relatorio HTML: `genhtml coverage/lcov.info -o coverage/html`.
