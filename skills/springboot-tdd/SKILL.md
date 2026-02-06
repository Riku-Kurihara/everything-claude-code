---
name: springboot-tdd
description: JUnit 5、Mockito、MockMvc、Testcontainers、JaCoCo を使用した Spring Boot のテスト駆動開発。機能追加、バグ修正、リファクタリングの際に使用します。
---

# Spring Boot TDD ワークフロー

80% 以上のカバレッジ（ユニット + 統合）を備えた Spring Boot サービスの TDD ガイダンス。

## 使用時期

- 新機能またはエンドポイントの追加
- バグ修正またはリファクタリング
- データアクセスロジックまたはセキュリティルールの追加

## ワークフロー

1) テストを最初に記述（失敗するはず）
2) テストを通すための最小限のコードを実装
3) テストが成功した状態でリファクタリング
4) カバレッジを強制（JaCoCo）

## ユニットテスト（JUnit 5 + Mockito）

```java
@ExtendWith(MockitoExtension.class)
class MarketServiceTest {
  @Mock MarketRepository repo;
  @InjectMocks MarketService service;

  @Test
  void createsMarket() {
    CreateMarketRequest req = new CreateMarketRequest("name", "desc", Instant.now(), List.of("cat"));
    when(repo.save(any())).thenAnswer(inv -> inv.getArgument(0));

    Market result = service.create(req);

    assertThat(result.name()).isEqualTo("name");
    verify(repo).save(any());
  }
}
```

パターン：
- Arrange-Act-Assert
- 部分的なモックを避け、明示的なスタブを優先
- バリアントには `@ParameterizedTest` を使用

## ウェブレイヤーテスト（MockMvc）

```java
@WebMvcTest(MarketController.class)
class MarketControllerTest {
  @Autowired MockMvc mockMvc;
  @MockBean MarketService marketService;

  @Test
  void returnsMarkets() throws Exception {
    when(marketService.list(any())).thenReturn(Page.empty());

    mockMvc.perform(get("/api/markets"))
        .andExpect(status().isOk())
        .andExpect(jsonPath("$.content").isArray());
  }
}
```

## 統合テスト（SpringBootTest）

```java
@SpringBootTest
@AutoConfigureMockMvc
@ActiveProfiles("test")
class MarketIntegrationTest {
  @Autowired MockMvc mockMvc;

  @Test
  void createsMarket() throws Exception {
    mockMvc.perform(post("/api/markets")
        .contentType(MediaType.APPLICATION_JSON)
        .content("""
          {"name":"Test","description":"Desc","endDate":"2030-01-01T00:00:00Z","categories":["general"]}
        """))
      .andExpect(status().isCreated());
  }
}
```

## 永続化テスト（DataJpaTest）

```java
@DataJpaTest
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
@Import(TestContainersConfig.class)
class MarketRepositoryTest {
  @Autowired MarketRepository repo;

  @Test
  void savesAndFinds() {
    MarketEntity entity = new MarketEntity();
    entity.setName("Test");
    repo.save(entity);

    Optional<MarketEntity> found = repo.findByName("Test");
    assertThat(found).isPresent();
  }
}
```

## Testcontainers

- PostgreSQL/Redis の再利用可能なコンテナを使用して本番環境をミラーリング
- `@DynamicPropertySource` 経由で Spring コンテキストに JDBC URL を注入

## カバレッジ（JaCoCo）

Maven スニペット：
```xml
<plugin>
  <groupId>org.jacoco</groupId>
  <artifactId>jacoco-maven-plugin</artifactId>
  <version>0.8.14</version>
  <executions>
    <execution>
      <goals><goal>prepare-agent</goal></goals>
    </execution>
    <execution>
      <id>report</id>
      <phase>verify</phase>
      <goals><goal>report</goal></goals>
    </execution>
  </executions>
</plugin>
```

## アサーション

- 読みやすさのために AssertJ（`assertThat`）を優先
- JSON レスポンスの場合は `jsonPath` を使用
- 例外の場合：`assertThatThrownBy(...)`

## テストデータビルダー

```java
class MarketBuilder {
  private String name = "Test";
  MarketBuilder withName(String name) { this.name = name; return this; }
  Market build() { return new Market(null, name, MarketStatus.ACTIVE); }
}
```

## CI コマンド

- Maven: `mvn -T 4 test` または `mvn verify`
- Gradle: `./gradlew test jacocoTestReport`

**重要**: テストは高速で独立しており、決定論的であることが重要です。実装の詳細ではなく、動作をテストしましょう。
