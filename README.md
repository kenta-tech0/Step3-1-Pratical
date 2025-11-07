# 顧客管理システム (CRM) 技術学習まとめ

## 目次
- [プロジェクト概要](#プロジェクト概要)
- [技術スタック](#技術スタック)
- [システムアーキテクチャ](#システムアーキテクチャ)
- [フロントエンド学習](#フロントエンド学習)
- [バックエンド学習](#バックエンド学習)
- [データフロー](#データフロー)
- [重要な学び](#重要な学び)


---

## プロジェクト概要

顧客情報の作成・参照・更新・削除（CRUD操作）を行うWebアプリケーション

### 主な機能
- ✅ 顧客の新規登録
- ✅ 顧客情報の表示
- ✅ 顧客情報の編集
- ✅ 顧客の削除

---

## 技術スタック

### フロントエンド
| 技術 | バージョン | 用途 |
|------|-----------|------|
| Next.js | App Router | Reactフレームワーク |
| React | 19+ | UIライブラリ |
| Tailwind CSS | - | CSSフレームワーク |
| DaisyUI | - | UIコンポーネント |

### バックエンド
| 技術 | 用途 |
|------|------|
| FastAPI | Web APIフレームワーク |
| SQLAlchemy | ORM（データベース操作） |
| Pydantic | データバリデーション |
| SQLite | データベース |
| Pandas | データ処理 |

---

## システムアーキテクチャ

```
┌─────────────────────────────────────────────────────────────┐
│                        Frontend (Next.js)                    │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐   │
│  │  Create  │  │   Read   │  │  Update  │  │  Delete  │   │
│  │  Page    │  │   Page   │  │   Page   │  │   Page   │   │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘   │
│       │             │              │             │          │
│       └─────────────┴──────────────┴─────────────┘          │
│                         │                                    │
└─────────────────────────┼────────────────────────────────────┘
                          │ HTTP (CORS)
                          │
┌─────────────────────────┼────────────────────────────────────┐
│                         ▼                                     │
│                   FastAPI (app.py)                           │
│              ┌──────────────────────┐                        │
│              │  RESTful API Routes   │                       │
│              │  - POST /customers    │                       │
│              │  - GET /customers     │                       │
│              │  - PUT /customers     │                       │
│              │  - DELETE /customers  │                       │
│              └──────────┬────────────┘                       │
│                         │                                     │
│              ┌──────────▼────────────┐                       │
│              │   CRUD Layer (crud.py) │                      │
│              │  - myinsert()          │                      │
│              │  - myselect()          │                      │
│              │  - myupdate()          │                      │
│              │  - mydelete()          │                      │
│              └──────────┬────────────┘                       │
│                         │                                     │
│              ┌──────────▼────────────┐                       │
│              │  ORM Models            │                      │
│              │  (mymodels.py)         │                      │
│              │  - Customers           │                      │
│              │  - Items               │                      │
│              │  - Purchases           │                      │
│              │  - PurchaseDetails     │                      │
│              └──────────┬────────────┘                       │
│                         │                                     │
│              ┌──────────▼────────────┐                       │
│              │  SQLAlchemy Engine     │                      │
│              │  (connect.py)          │                      │
│              └──────────┬────────────┘                       │
│                         │                                     │
└─────────────────────────┼────────────────────────────────────┘
                          │
                          ▼
                   ┌─────────────┐
                   │  SQLite DB  │
                   │  (CRM.db)   │
                   └─────────────┘
```

---

## フロントエンド学習

### 1. Next.js App Router の使い方

#### Client Component の宣言
```javascript
"use client";
```
- ブラウザ側でのみ実行されるコンポーネント
- `useState`, `useEffect`などのReact Hooksを使用可能
- イベントハンドラーを持つコンポーネントに必須

#### 動的ルーティングのパラメータ取得
```javascript
export default function Page(props) {
  const params = use(props.params);  // React 19の新しいAPI
  const id = params.id;
}
```

### 2. フォーム処理パターン

#### useRefを使ったフォーム管理
```javascript
const formRef = useRef();

const handleSubmit = async (event) => {
  event.preventDefault();
  const formData = new FormData(formRef.current);
  const customerID = formData.get("customer_id");
};
```

**学び：**
- `FormData` APIで簡単にフォームデータを取得
- `useRef`でDOM要素への直接アクセス
- `event.preventDefault()`でデフォルト送信を防止

#### バリデーション実装
```javascript
if (!customerID) {
  alert("CustomerIDは必須です!空白は許可されません!");
  return;
}
```

### 3. データフェッチとState管理

#### 非同期データ取得パターン
```javascript
const [customer, setCustomer] = useState(null);

useEffect(() => {
  const fetchAndSetCustomer = async () => {
    const customerInfo = await fetchCustomer(customer_id);
    setCustomer(customerInfo);
  };
  fetchAndSetCustomer();
}, []); // 依存配列の注意点！
```

**重要な学び：**
- `useEffect`の依存配列が空の場合、マウント時のみ実行
- 実際は`[customer_id]`を指定すべき（ESLint警告）
- 非同期関数は`useEffect`内で直接使えないため、内部で定義

### 4. Hydration エラーの回避

#### Dynamic Import でSSRを無効化
```javascript
import dynamic from "next/dynamic";

const OneCustomerInfoCard = dynamic(
  () => import("@/app/components/one_customer_info_card.jsx"),
  { ssr: false }
);
```

**なぜ必要か：**
- サーバー側の初期HTML ≠ クライアント側の最終HTML
- `useEffect`で取得したデータはサーバー側で存在しない
- SSRを無効にしてクライアントのみでレンダリング


### 5. ルーティングとナビゲーション

```javascript
import { useRouter } from "next/navigation";

const router = useRouter();
router.push(`./create/confirm?customer_id=${customerID}`);
```

**学び：**
- プログラマティックなページ遷移
- クエリパラメータの受け渡し
- 確認画面への遷移パターン

---

## バックエンド学習

### 1. FastAPI の基本構造

#### アプリケーションセットアップ
```python
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware

app = FastAPI()

app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],  # 本番環境では制限すべき！　
    allow_methods=["*"],
    allow_headers=["*"],
)
```

**学び：**
- CORSは開発時に必須（フロントエンドとバックエンドの分離）
- `allow_origins=["*"]`は開発用、本番は特定ドメインのみ許可

#### Pydantic によるバリデーション
```python
from pydantic import BaseModel

class Customer(BaseModel):
    customer_id: str
    customer_name: str
    age: int
    gender: str
```

**Pydanticの利点：**
- 自動的な型チェック
- バリデーションエラーの自動生成
- OpenAPI仕様の自動生成
- IDEの補完サポート

### 2. RESTful API設計

#### エンドポイント設計
```python
@app.post("/customers")      # 作成
@app.get("/customers")       # 単一取得
@app.get("/allcustomers")    # 全件取得
@app.put("/customers")       # 更新
@app.delete("/customers")    # 削除
```

**HTTPメソッドの使い分け：**
- `POST`: 新規作成（冪等性なし）
- `GET`: データ取得（安全）
- `PUT`: 全体更新（冪等性あり）
- `DELETE`: 削除（冪等性あり）

#### クエリパラメータの扱い
```python
@app.get("/customers")
def read_one_customer(customer_id: str = Query(...)):
    # Query(...)は必須パラメータを意味する
```

### 3. SQLAlchemy ORM

#### モデル定義（型アノテーション方式）
```python
from sqlalchemy.orm import DeclarativeBase, Mapped, mapped_column

class Base(DeclarativeBase):
    pass

class Customers(Base):
    __tablename__ = 'customers'
    customer_id: Mapped[str] = mapped_column(primary_key=True, nullable=False)
    customer_name: Mapped[str] = mapped_column()
    age: Mapped[int] = mapped_column()
    gender: Mapped[str] = mapped_column()
```

**学び：**
- `Mapped[型]`で型安全性を確保
- `mapped_column()`でカラム属性を定義
- Pythonクラス ↔ DBテーブル の自動マッピング

#### 外部キー制約
```python
class Purchases(Base):
    purchase_name: Mapped[str] = mapped_column(
        ForeignKey("customers.customer_id") →　これが制約
    )
```

#### 複合主キー
```python
class PurchaseDetails(Base):
    purchase_id: Mapped[int] = mapped_column(
        ForeignKey("purchases.purchase_id"), 
        primary_key=True
    )
    item_name: Mapped[str] = mapped_column(
        ForeignKey("items.item_id"), 
        primary_key=True
    )
```

### 4. データベース操作パターン

#### セッション管理の基本形
```python
from sqlalchemy.orm import sessionmaker

Session = sessionmaker(bind=engine)
session = Session()

try:
    with session.begin():  # トランザクション開始
        # データベース操作
        session.execute(query)
except sqlalchemy.exc.IntegrityError:
    session.rollback()
finally:
    session.close()  # 必ずクローズ
```

**学び：**
- トランザクション管理の重要性
- 例外発生時の自動ロールバック
- リソースの確実な解放

#### CRUD操作の実装

**INSERT（作成）**
```python
def myinsert(mymodel, values):
    query = insert(mymodel).values(values)
    session.execute(query)
```

**SELECT（取得）**
```python
# 単一取得
query = session.query(mymodel).filter(mymodel.customer_id == customer_id)
result = query.all()

# 全件取得（Pandas使用）
query = select(mymodel)
df = pd.read_sql_query(query, con=engine)
result_json = df.to_json(orient='records', force_ascii=False)
```

**UPDATE（更新）**
```python
customer_id = values.pop("customer_id")  # WHERE句用に分離
query = update(mymodel).where(
    mymodel.customer_id == customer_id
).values(values)
```

**DELETE（削除）**
```python
query = delete(mymodel).where(mymodel.customer_id == customer_id)
```

### 5. データ変換パターン

#### ORM → JSON変換
```python
# 方法1: 手動で辞書化
result_dict_list = []
for customer in result:
    result_dict_list.append({
        "customer_id": customer.customer_id,
        "customer_name": customer.customer_name,
        "age": customer.age,
        "gender": customer.gender
    })
result_json = json.dumps(result_dict_list, ensure_ascii=False)

# 方法2: Pandas使用（大量データに適している）
df = pd.read_sql_query(query, con=engine)
result_json = df.to_json(orient='records', force_ascii=False)
```

**`ensure_ascii=False` / `force_ascii=False`の重要性：**
- 日本語を正しくエンコード
- これがないと文字化けが発生

---

## データフロー

### 作成（CREATE）処理の詳細フロー

```
1. ユーザーがフォームに入力
   ↓
2. [Frontend] handleSubmit実行
   - event.preventDefault()
   - FormDataでデータ収集
   - バリデーション
   ↓
3. [Frontend] createCustomer(formData)
   - Next.js Server Actionまたはfetch呼び出し
   ↓
4. [Backend] POST /customers エンドポイント
   - Pydanticで自動バリデーション
   - Customer型に変換
   ↓
5. [Backend] crud.myinsert()
   - SQLAlchemyセッション作成
   - INSERT文実行
   - トランザクションコミット
   ↓
6. [Database] SQLiteにデータ保存
   ↓
7. [Backend] crud.myselect()
   - 作成したデータを再取得（確認用）
   ↓
8. [Backend] JSON形式でレスポンス
   ↓
9. [Frontend] router.push()
   - 確認画面へリダイレクト
   ↓
10. [Frontend] 確認画面表示
```

### 更新（UPDATE）処理の注意点

```python
# app.pyでvaluesをコピーする理由
values = customer.dict()
values_original = values.copy()  # ← これが重要！

crud.myupdate(mymodels.Customers, values)
# ↑ この中でvalues.pop("customer_id")が実行される

# customer_idが消えているので、オリジナルを使用
crud.myselect(mymodels.Customers, values_original.get("customer_id"))
```

**学び：**
- `dict.pop()`は破壊的メソッド
- 関数内での変更が呼び出し元に影響
- 必要なデータは事前にコピー

---

## 重要な学び

### 1. フロントエンド開発

#### ✅ 理解したこと
- **Client vs Server Components**: いつどちらを使うべきか
- **フォーム処理**: FormData APIの便利さ
- **Hydration エラー**: 原因と対処法
- **動的ルーティング**: パラメータの受け渡し

#### ⚠️ 注意が必要なこと
- SSRとCSRの違いを意識する
- 非同期処理のエラーハンドリング
- useEffectの無限ループに注意
- メモリリークの防止（cleanup関数）

### 2. バックエンド開発

#### ✅ 理解したこと
- **ORM の利点**: SQLを直接書かない安全性
- **トランザクション管理**: ACID特性の保証
- **REST API設計**: リソース指向の考え方
- **バリデーション**: Pydanticによる型安全性
- **CORS**: クロスオリジン通信の仕組み

#### ⚠️ 注意が必要なこと
- セッションのクローズ忘れ（リソースリーク）
- トランザクションのロールバック
- 一意制約違反のハンドリング
- 大量データのパフォーマンス

### 3. アーキテクチャ設計

#### レイヤー分離の重要性
```
Presentation Layer (UI)
    ↓
API Layer (FastAPI routes)
    ↓
Business Logic Layer (CRUD functions)
    ↓
Data Access Layer (ORM)
    ↓
Database Layer (SQLite)
```

**メリット：**
- 各層の責務が明確
- テストしやすい
- 変更の影響範囲が限定的
- 再利用性が高い

---

### バックエンド

#### 1. エラーレスポンスの統一
```python
from fastapi import HTTPException

@app.post("/customers")
def create_customer(customer: Customer):
    try:
        crud.myinsert(mymodels.Customers, customer.dict())
    except sqlalchemy.exc.IntegrityError:
        raise HTTPException(
            status_code=409,
            detail="Customer ID already exists"
        )
    except Exception as e:
        raise HTTPException(
            status_code=500,
            detail=f"Internal server error: {str(e)}"
        )
```

#### 2. セッション管理の改善
```python
from contextlib import contextmanager

@contextmanager
def get_db_session():
    Session = sessionmaker(bind=engine)
    session = Session()
    try:
        yield session
        session.commit()
    except Exception:
        session.rollback()
        raise
    finally:
        session.close()

# 使用例
def myinsert(mymodel, values):
    with get_db_session() as session:
        query = insert(mymodel).values(values)
        session.execute(query)
```

#### 3. ページネーション実装
```python
@app.get("/allcustomers")
def read_all_customer(
    skip: int = Query(0, ge=0),
    limit: int = Query(100, ge=1, le=1000)
):
    query = select(Customers).offset(skip).limit(limit)
    # 実装...
```

#### 4. ログ管理
```python
import logging

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

@app.post("/customers")
def create_customer(customer: Customer):
    logger.info(f"Creating customer: {customer.customer_id}")
    try:
        result = crud.myinsert(mymodels.Customers, customer.dict())
        logger.info(f"Successfully created customer: {customer.customer_id}")
        return result
    except Exception as e:
        logger.error(f"Failed to create customer: {str(e)}")
        raise
```

#### 5. 環境変数の活用
```python
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    database_url: str
    cors_origins: list[str]
    
    class Config:
        env_file = ".env"

settings = Settings()

app.add_middleware(
    CORSMiddleware,
    allow_origins=settings.cors_origins,  # ["http://localhost:3000"]
)
```

### セキュリティ

#### 1. 認証・認可の追加
```python
from fastapi.security import HTTPBearer

security = HTTPBearer()

@app.post("/customers")
async def create_customer(
    customer: Customer,
    credentials: HTTPAuthorizationCredentials = Depends(security)
):
    # トークン検証
    verify_token(credentials.credentials)
    # 処理続行...
```

#### 2. レート制限
```python
from slowapi import Limiter
from slowapi.util import get_remote_address

limiter = Limiter(key_func=get_remote_address)

@app.post("/customers")
@limiter.limit("5/minute")
def create_customer(request: Request, customer: Customer):
    # 1分間に5回まで
```

#### 3. 入力のサニタイゼーション
```python
from html import escape

class Customer(BaseModel):
    customer_name: str
    
    @validator('customer_name')
    def sanitize_name(cls, v):
        return escape(v)  # XSS対策
```

---

## プロダクトマネージャーとしての視点

### 技術的負債の管理

#### 優先度：高
1. **エラーハンドリングの実装** - ユーザー体験に直結
2. **CORS設定の本番対応** - セキュリティリスク
3. **文字化け修正** - 基本的な品質問題

#### 優先度：中
1. **ローディング状態の表示** - UX改善
2. **バリデーション強化** - データ品質向上
3. **ログ管理の導入** - 運用保守性向上

#### 優先度：低（将来対応）
1. **認証・認可機能** - 機能追加フェーズで
2. **ページネーション** - データ量増加時
3. **レート制限** - スケール時に検討

### 開発プロセスの改善

#### コードレビューのポイント
- [ ] エラーハンドリングは適切か
- [ ] セキュリティリスクはないか
- [ ] パフォーマンス問題はないか
- [ ] テストは書かれているか
- [ ] ドキュメントは更新されているか

#### テスト戦略
```
Unit Tests (単体テスト)
├── Frontend: コンポーネントテスト（Jest + React Testing Library）
└── Backend: 関数テスト（pytest）

Integration Tests (結合テスト)
├── API エンドポイントテスト
└── データベーストランザクションテスト

E2E Tests (エンドツーエンドテスト)
└── ユーザーシナリオテスト（Playwright / Cypress）
```

---

## まとめ

### 学習成果
このプロジェクトを通じて、以下の技術スタックを学習しました：

**フロントエンド:**
- Next.js App Routerの基本
- React Hooksを使った状態管理
- フォーム処理とバリデーション
- CSRとSSRの違い
- APIとの通信

**バックエンド:**
- FastAPIによるREST API構築
- SQLAlchemy ORMの使用
- データベース設計（ER図）
- トランザクション管理
- CRUD操作の実装

**アーキテクチャ:**
- レイヤー分離の設計
- フロントエンドとバックエンドの分離
- RESTful APIの設計原則



---

## 参考リソース

### 公式ドキュメント
- [Next.js Documentation](https://nextjs.org/docs)
- [FastAPI Documentation](https://fastapi.tiangolo.com/)
- [SQLAlchemy Documentation](https://docs.sqlalchemy.org/)
- [React Documentation](https://react.dev/)

### 学習リソース
- Next.js チュートリアル
- FastAPI チュートリアル
- SQLAlchemy ORM チュートリアル
- REST API設計のベストプラクティス

---

**作成日**: 2025年11月7日  
**バージョン**: 1.0  
**作成者**: Kenta Funaki
