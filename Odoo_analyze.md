# Odoo ERP 개발 분석 문서

> **작성일**: 2024-12-02  
> **목적**: Odoo 개발/커스터마이징 직무 면접 준비 및 학습  
> **실습 방향**: HR → 재고관리 → 영업  

---
## 목차

1. [Odoo 아키텍처 & 사용법](#1-odoo-아키텍처--사용법)
2. [OWL vs React 비교 (프론트엔드)](#2-owl-vs-react-비교-프론트엔드)
3. [기술 선택 이유 (PostgreSQL, RPC, XML)](#3-기술-선택-이유-postgresql-rpc-xml)
4. [Community vs Enterprise 비교](#4-community-vs-enterprise-비교)
5. [Odoo 17 vs Odoo 18 비교](#5-odoo-17-vs-odoo-18-비교)
6. [커스터마이징 방식](#6-커스터마이징-방식)
7. [데이터 Import/Migration](#7-데이터-importmigration)
8. [PM & 개발 워크플로우](#8-pm--개발-워크플로우)
9. [개발 환경 선택](#9-개발-환경-선택)
10. [학습 로드맵](#10-학습-로드맵)

---

## 1. Odoo 아키텍처 & 사용법

### 1.1 3-Tier 아키텍처

| 레이어 | 기술 | 설명 |
|--------|------|------|
| **Presentation** | JavaScript/OWL, XML, QWeb | 웹 클라이언트, 뷰, 템플릿 |
| **Business Logic** | Python | 모델, ORM, 컨트롤러 |
| **Data** | PostgreSQL | 데이터 저장 (필수 - 대체 불가) |

### 1.2 기술 스택

- **Backend**: Python 3.10+
- **Database**: PostgreSQL (필수)
- **Frontend**: JavaScript + OWL (Odoo Web Library - React 유사 컴포넌트 프레임워크)
- **Templating**: QWeb (XML 기반 템플릿 엔진)
- **API Protocol**: JSON-RPC, XML-RPC

### 1.3 ORM (Object-Relational Mapping)

#### 모델 타입
```python
models.Model          # 일반 영구 모델 (DB 저장)
models.TransientModel # 임시 레코드 (자동 삭제)
models.AbstractModel  # 인스턴스화 불가, 상속용
```

#### 필드 타입
```python
# 기본 필드
fields.Char, fields.Text, fields.Integer, fields.Float
fields.Boolean, fields.Date, fields.Datetime, fields.Binary, fields.Selection

# 관계 필드
fields.Many2one      # N:1 관계
fields.One2many      # 1:N 관계 (역참조)
fields.Many2many     # N:M 관계
```

#### 핵심 ORM 메서드
```python
self.env['model.name'].create(vals)      # 레코드 생성
self.env['model.name'].search(domain)    # 레코드 검색
recordset.write(vals)                     # 레코드 수정
recordset.unlink()                        # 레코드 삭제
recordset.read(['field1', 'field2'])     # 특정 필드 읽기
```

### 1.4 모듈 구조

```
my_module/
├── __init__.py           # Python 패키지 초기화
├── __manifest__.py       # 모듈 메타데이터 (핵심!)
├── models/               # Python 모델 파일
│   ├── __init__.py
│   └── my_model.py
├── views/                # XML 뷰 정의
│   └── my_views.xml
├── security/             # 접근 권한
│   ├── ir.model.access.csv
│   └── security.xml
├── data/                 # 기본 데이터
├── static/               # 웹 에셋 (JS, CSS, 이미지)
├── controllers/          # HTTP 컨트롤러
├── wizards/              # 마법사용 임시 모델
└── reports/              # 리포트 템플릿
```

### 1.5 Manifest 파일 (`__manifest__.py`)

```python
{
    'name': 'Module Name',
    'version': '17.0.1.0.0',
    'category': 'Human Resources',
    'summary': '간단한 설명',
    'description': '상세 설명',
    'author': 'Your Name',
    'website': 'https://example.com',
    'license': 'LGPL-3',
    'depends': ['base', 'hr'],      # 의존성 모듈
    'data': [
        'security/ir.model.access.csv',
        'views/my_views.xml',
    ],
    'demo': ['demo/demo_data.xml'],
    'application': True,             # 메인 앱 여부
    'installable': True,
    'auto_install': False,
}
```

---

## 2. OWL vs React 비교 (프론트엔드)

> **배경**: React/TypeScript/Next.js 경험자 관점에서 OWL 이해하기

### 2.1 OWL (Odoo Web Library) 개요

OWL은 Odoo 14부터 도입된 **React 영감을 받은 컴포넌트 프레임워크**입니다.  
기존 Odoo의 jQuery 기반 위젯 시스템을 대체하며, 현대적인 반응형 UI 개발을 지원합니다.

### 2.2 OWL vs React 핵심 비교

| 항목 | React | OWL |
|------|-------|-----|
| **개발사** | Meta (Facebook) | Odoo SA |
| **언어** | JSX (JavaScript + XML) | JavaScript + XML 템플릿 |
| **템플릿** | JSX (코드 내 inline) | QWeb XML (별도 파일) |
| **상태 관리** | useState, useReducer, Redux 등 | useState, reactive (내장) |
| **생명주기** | useEffect 등 Hooks | onMounted, onWillUnmount 등 |
| **Virtual DOM** | ✅ 있음 | ✅ 있음 |
| **생태계** | 거대함 (npm 패키지 무수히 많음) | Odoo 전용 (제한적) |
| **타입 지원** | TypeScript 완벽 지원 | 제한적 (타입 없음) |

### 2.3 컴포넌트 구조 비교

#### React 컴포넌트
```jsx
// React (JSX)
import { useState, useEffect } from 'react';

function Counter() {
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    console.log('Mounted');
    return () => console.log('Unmounted');
  }, []);
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>+1</button>
    </div>
  );
}
```

#### OWL 컴포넌트
```javascript
// OWL (JavaScript + XML 템플릿)
import { Component, useState, onMounted, onWillUnmount, xml } from "@odoo/owl";

class Counter extends Component {
  static template = xml`
    <div>
      <p>Count: <t t-esc="state.count"/></p>
      <button t-on-click="increment">+1</button>
    </div>
  `;
  
  setup() {
    this.state = useState({ count: 0 });
    
    onMounted(() => console.log('Mounted'));
    onWillUnmount(() => console.log('Unmounted'));
  }
  
  increment() {
    this.state.count++;
  }
}
```

### 2.4 Hooks 비교

| React Hook | OWL 대응 | 설명 |
|------------|----------|------|
| `useState` | `useState` | 상태 관리 (거의 동일!) |
| `useEffect` | `onMounted`, `onWillUnmount`, `onPatched` | 생명주기 분리됨 |
| `useRef` | `useRef` | DOM 참조 (동일 개념) |
| `useContext` | `useEnv` | 환경/컨텍스트 접근 |
| `useMemo` | - | 직접 구현 필요 |
| `useCallback` | - | 직접 구현 필요 |
| Custom Hooks | Custom Hooks | ✅ 지원 (Odoo 18 개선) |

### 2.5 상태 관리 비교

#### React (useState + Context)
```jsx
// React - 상태가 변하면 리렌더링
const [user, setUser] = useState({ name: 'Kim', age: 25 });
setUser({ ...user, age: 26 }); // immutable update 필요
```

#### OWL (reactive)
```javascript
// OWL - Proxy 기반 반응성 (Vue와 유사)
import { useState } from "@odoo/owl";

setup() {
  this.state = useState({ name: 'Kim', age: 25 });
}

updateAge() {
  this.state.age = 26; // 직접 수정 가능! (Vue처럼)
}
```

**핵심 차이**: OWL은 **Vue의 반응성 시스템**과 유사하게 Proxy를 사용하여 객체를 직접 수정해도 자동으로 변경을 감지합니다. React처럼 불변성(immutability)을 유지할 필요가 없습니다.

### 2.6 템플릿 문법 (QWeb)

QWeb은 XML 기반 템플릿 엔진으로, **Vue의 template**과 매우 유사합니다:

```xml
<!-- QWeb 템플릿 (Vue 템플릿과 비교) -->
<t t-if="state.isLoading">
  <p>Loading...</p>
</t>
<t t-elif="state.error">
  <p t-esc="state.error"/>
</t>
<t t-else="">
  <ul>
    <t t-foreach="state.items" t-as="item" t-key="item.id">
      <li t-esc="item.name"/>
    </t>
  </ul>
</t>

<!-- 이벤트 바인딩 -->
<button t-on-click="handleClick">Click</button>
<input t-on-input="handleInput" t-att-value="state.value"/>

<!-- 조건부 속성 -->
<div t-att-class="{ 'active': state.isActive, 'disabled': !state.isEnabled }"/>
```

| QWeb 지시자 | Vue 대응 | React 대응 |
|-------------|----------|------------|
| `t-if`, `t-elif`, `t-else` | `v-if`, `v-else-if`, `v-else` | 삼항 연산자, && |
| `t-foreach`, `t-as` | `v-for` | `.map()` |
| `t-on-click` | `@click` | `onClick` |
| `t-att-{attr}` | `:attr` | `{attr}` |
| `t-esc` | `{{ }}` | `{}` |

### 2.7 React 개발자가 OWL에서 주의할 점

1. **JSX 없음**: 템플릿은 XML로 별도 정의 (적응 필요)
2. **TypeScript 미지원**: 타입 안정성 없음 (불편할 수 있음)
3. **제한된 생태계**: npm 패키지 사용 어려움, Odoo 전용
4. **class 기반**: 함수형 컴포넌트 없음 (class + setup 패턴)
5. **Odoo 종속**: 독립 실행 불가, Odoo 환경에서만 동작

### 2.8 장점 (React 대비)

1. **Odoo 통합**: ORM, RPC, 서비스와 자연스럽게 연동
2. **Vue 스타일 반응성**: 직접 상태 수정 가능 (간편)
3. **XML 분리**: 로직과 뷰 분리 (관심사 분리)
4. **내장 서비스**: notification, dialog, orm 등 바로 사용

---

## 3. 기술 선택 이유 (PostgreSQL, RPC, XML)

### 3.1 PostgreSQL이 필수인 이유

> **질문**: MySQL, SQLite 등 다른 DB는 왜 안 되나요?

#### Odoo가 PostgreSQL만 지원하는 이유

| 기능 | PostgreSQL | MySQL | SQLite |
|------|------------|-------|--------|
| **JSONB 지원** | ✅ 네이티브 | ⚠️ 제한적 | ❌ |
| **Array 타입** | ✅ | ❌ | ❌ |
| **Window Functions** | ✅ 완전 | ⚠️ 부분 | ⚠️ 부분 |
| **CTE (WITH 절)** | ✅ 완전 | ⚠️ MySQL 8+ | ⚠️ 제한 |
| **Full Text Search** | ✅ 내장 | ✅ 내장 | ❌ |
| **ACID 준수** | ✅ 엄격 | ⚠️ InnoDB만 | ⚠️ |
| **동시성 처리** | ✅ MVCC | ⚠️ 락 기반 | ❌ 단일 |

#### Odoo가 PostgreSQL 전용 기능을 사용하는 부분

```sql
-- 1. JSONB 필드 (Odoo의 many2many 태그 등)
SELECT * FROM model WHERE tags::jsonb ? 'important';

-- 2. Array 타입 (다중 선택 필드)
SELECT * FROM model WHERE category_ids && ARRAY[1, 2, 3];

-- 3. Window Functions (리포트, 분석)
SELECT name, SUM(amount) OVER (PARTITION BY partner_id) 
FROM account_move;

-- 4. RETURNING 절 (INSERT 후 ID 즉시 반환)
INSERT INTO res_partner (name) VALUES ('Test') RETURNING id;

-- 5. UPSERT (ON CONFLICT)
INSERT INTO model (key, value) VALUES ('a', 1)
ON CONFLICT (key) DO UPDATE SET value = EXCLUDED.value;
```

#### 결론
Odoo의 ORM은 PostgreSQL 전용 SQL을 생성합니다. **다른 DB로 변경 시 ORM 전체 재작성이 필요**하므로, PostgreSQL은 사실상 필수입니다.

### 3.2 JSON-RPC vs XML-RPC vs gRPC 비교

> **질문**: REST JSON이나 gRPC를 많이 썼는데, RPC 방식의 차이가 뭔가요?

#### 프로토콜 비교

| 항목 | REST JSON | JSON-RPC | XML-RPC | gRPC |
|------|-----------|----------|---------|------|
| **포맷** | JSON | JSON | XML | Protocol Buffers |
| **전송** | HTTP | HTTP | HTTP | HTTP/2 |
| **타입 정의** | OpenAPI (선택) | JSON Schema (선택) | DTD (선택) | .proto (필수) |
| **성능** | 좋음 | 좋음 | 느림 | 매우 빠름 |
| **가독성** | ✅ 좋음 | ✅ 좋음 | ⚠️ verbose | ❌ 바이너리 |
| **브라우저** | ✅ | ✅ | ✅ | ⚠️ grpc-web 필요 |

#### 실제 요청/응답 비교

```python
# === REST JSON (일반적인 방식) ===
# POST /api/partners
# Body: {"name": "Test", "email": "test@test.com"}
# Response: {"id": 1, "name": "Test", ...}

# === JSON-RPC (Odoo 방식) ===
# POST /jsonrpc
{
    "jsonrpc": "2.0",
    "method": "call",
    "params": {
        "service": "object",
        "method": "execute_kw",
        "args": ["mydb", 1, "password", "res.partner", "create", 
                 [{"name": "Test"}]]
    },
    "id": 1
}
# Response:
{
    "jsonrpc": "2.0",
    "result": 42,  # 생성된 ID
    "id": 1
}

# === XML-RPC (Odoo 레거시 방식) ===
# POST /xmlrpc/2/object
<?xml version="1.0"?>
<methodCall>
  <methodName>execute_kw</methodName>
  <params>
    <param><value><string>mydb</string></value></param>
    <param><value><int>1</int></value></param>
    <param><value><string>password</string></value></param>
    <param><value><string>res.partner</string></value></param>
    <param><value><string>create</string></value></param>
    <param><value><array><data>
      <value><struct>
        <member><name>name</name><value><string>Test</string></value></member>
      </struct></value>
    </data></array></value></param>
  </params>
</methodCall>
```

#### Odoo에서 어떤 것을 사용할까?

| 용도 | 권장 프로토콜 | 이유 |
|------|--------------|------|
| **외부 스크립트** | JSON-RPC | 간결, Python 라이브러리 지원 |
| **레거시 통합** | XML-RPC | 오래된 시스템 호환 |
| **웹 클라이언트** | JSON-RPC | 브라우저 네이티브 지원 |
| **고성능 필요** | REST + JSON | 커스텀 컨트롤러 구현 |

**Tip**: Odoo에서 REST API가 필요하면 직접 컨트롤러를 만들 수 있습니다:

```python
from odoo import http

class MyAPI(http.Controller):
    @http.route('/api/partners', type='json', auth='user', methods=['POST'])
    def create_partner(self, **kwargs):
        partner = http.request.env['res.partner'].create(kwargs)
        return {'id': partner.id, 'name': partner.name}
```

### 3.3 XML의 역할 (Java 느낌이 나는 이유)

> **질문**: XML을 Java에서나 써봤는데, Odoo에서는 왜 XML을 쓰나요?

#### Odoo에서 XML이 사용되는 곳

| 용도 | 파일 | 설명 |
|------|------|------|
| **뷰 정의** | `views/*.xml` | Form, Tree, Kanban 등 UI 정의 |
| **메뉴 정의** | `views/menu.xml` | 네비게이션 메뉴 |
| **액션 정의** | `views/actions.xml` | 윈도우 액션, 서버 액션 |
| **보안 규칙** | `security/security.xml` | Record Rules |
| **데이터** | `data/*.xml` | 초기 데이터, 설정값 |
| **리포트** | `reports/*.xml` | QWeb 리포트 템플릿 |
| **QWeb 템플릿** | `static/src/xml/*.xml` | 프론트엔드 템플릿 |

#### Django vs Odoo 비교

```python
# === Django 방식 (Python + HTML 템플릿) ===

# models.py
class Partner(models.Model):
    name = models.CharField(max_length=100)

# views.py (Python)
def partner_list(request):
    partners = Partner.objects.all()
    return render(request, 'partner_list.html', {'partners': partners})

# templates/partner_list.html (HTML + Jinja)
{% for partner in partners %}
  <div>{{ partner.name }}</div>
{% endfor %}
```

```python
# === Odoo 방식 (Python + XML) ===

# models/partner.py
class Partner(models.Model):
    _name = 'res.partner'
    name = fields.Char()

# views/partner_views.xml (XML이 뷰 역할!)
<record id="view_partner_tree" model="ir.ui.view">
    <field name="name">partner.tree</field>
    <field name="model">res.partner</field>
    <field name="arch" type="xml">
        <tree>
            <field name="name"/>
        </tree>
    </field>
</record>
```

#### XML을 쓰는 이유

1. **선언적(Declarative)**: UI를 코드가 아닌 데이터로 정의
2. **데이터베이스 저장**: XML이 DB에 저장되어 런타임 수정 가능
3. **상속 용이**: XPath로 기존 뷰를 정밀하게 수정
4. **노코드 호환**: Studio에서 XML을 자동 생성
5. **역사적 이유**: Odoo 초기(TinyERP)부터 이 방식 사용

#### Java와의 유사점/차이점

| 항목 | Java (Spring) | Odoo |
|------|--------------|------|
| **설정 파일** | XML 또는 어노테이션 | XML (Manifest + Views) |
| **의존성 주입** | Spring XML/Bean | `_inherit` 상속 |
| **ORM 매핑** | Hibernate XML/JPA | Python 클래스 |
| **뷰 정의** | JSP/Thymeleaf | QWeb XML |

**결론**: Odoo의 XML 사용은 Java 엔터프라이즈 패턴의 영향을 받았지만, **뷰와 데이터를 코드가 아닌 레코드로 관리**하는 독특한 설계입니다.

### 3.4 CSV 파일의 역할 (`ir.model.access.csv`)

> **질문**: 왜 보안 설정에 CSV를 쓰나요?

#### CSV 파일 구조
```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_partner_user,res.partner.user,model_res_partner,base.group_user,1,1,1,0
access_partner_manager,res.partner.manager,model_res_partner,base.group_system,1,1,1,1
```

| 컬럼 | 설명 |
|------|------|
| `id` | 레코드 XML ID |
| `name` | 규칙 이름 |
| `model_id:id` | 대상 모델 (`model_` + `_` 변환된 모델명) |
| `group_id:id` | 적용 그룹 |
| `perm_read/write/create/unlink` | CRUD 권한 (1=허용, 0=거부) |

#### CSV를 쓰는 이유

1. **간결함**: 행렬 형태 데이터에 XML보다 효율적
2. **가독성**: 스프레드시트로 편집 가능
3. **일괄 처리**: 대량 권한 설정에 용이
4. **역사적 이유**: 초기 Odoo에서 권한 관리용으로 채택

**참고**: 복잡한 Record Rules는 XML로 정의합니다:
```xml
<!-- security/security.xml -->
<record id="rule_partner_own_company" model="ir.rule">
    <field name="name">Partners: own company only</field>
    <field name="model_id" ref="model_res_partner"/>
    <field name="domain_force">[('company_id', '=', user.company_id.id)]</field>
</record>
```

### 3.5 XPath를 쓰는 이유 (CSS 선택자 대신)

> **질문**: CSS 선택자 대신 XPath를 쓰는 이유가 뭔가요?

#### XPath vs CSS 선택자

| 항목 | CSS 선택자 | XPath |
|------|------------|-------|
| **대상** | HTML 요소 | XML/HTML 모든 노드 |
| **방향** | 아래로만 탐색 | 위/아래/옆 모두 가능 |
| **텍스트 선택** | ❌ 불가 | ✅ `text()` |
| **조건** | 제한적 (`:nth-child`) | 강력 (`[position() > 2]`) |
| **속성 기반** | `[name="value"]` | `[@name='value']` |

#### Odoo에서 XPath를 쓰는 이유

```xml
<!-- 1. 속성 기반 정밀 선택 -->
<xpath expr="//field[@name='email']" position="after">
    <field name="phone"/>
</xpath>

<!-- 2. 부모 요소 탐색 (CSS로 불가능!) -->
<xpath expr="//field[@name='name']/parent::group" position="inside">
    <field name="code"/>
</xpath>

<!-- 3. 텍스트 내용 기반 선택 -->
<xpath expr="//label[text()='Customer']" position="replace">
    <label string="Client"/>
</xpath>

<!-- 4. 형제 요소 탐색 -->
<xpath expr="//field[@name='date']/following-sibling::field[1]" position="replace">
    <field name="new_field"/>
</xpath>
```

**결론**: Odoo 뷰는 복잡한 중첩 구조의 XML이므로, 상하좌우 모든 방향 탐색이 가능한 XPath가 필수입니다.

### 3.6 Alembic을 사용하지 않는 이유

> **질문**: FastAPI/Django처럼 Alembic으로 마이그레이션 안 하나요?

#### Django/FastAPI vs Odoo 마이그레이션 비교

| 항목 | Django/Alembic | Odoo |
|------|---------------|------|
| **마이그레이션 파일** | Python 파일 생성 | ❌ 없음 (자동) |
| **스키마 관리** | 수동 (`makemigrations`) | 자동 (모듈 업그레이드) |
| **버전 추적** | 마이그레이션 히스토리 테이블 | `ir_module_module` 테이블 |
| **롤백** | 지원 (`migrate --backward`) | ❌ 미지원 (백업 필요) |

#### Odoo의 자동 마이그레이션

```python
# 필드 추가하면 자동으로 DB 컬럼 생성
class Partner(models.Model):
    _inherit = 'res.partner'
    new_field = fields.Char('New Field')  # 모듈 업그레이드 시 자동 추가!
```

```bash
# Odoo 업그레이드 명령
odoo-bin -u my_module -d mydb  # 이것만으로 스키마 자동 변경
```

#### Alembic이 필요 없는 이유

1. **ORM이 스키마 관리**: 모델 정의 = 스키마 정의
2. **업그레이드 시 자동 적용**: `-u` 옵션으로 컬럼 자동 추가/수정
3. **모듈 단위 관리**: 각 모듈이 자체 스키마 책임

#### 복잡한 마이그레이션이 필요할 때

```python
# __manifest__.py
{
    'pre_init_hook': 'pre_init_hook',
    'post_init_hook': 'post_init_hook',
}

# __init__.py
def pre_init_hook(cr):
    """모듈 설치 전 실행 (raw SQL 가능)"""
    cr.execute("UPDATE res_partner SET active = TRUE WHERE active IS NULL")

def post_init_hook(cr, registry):
    """모듈 설치 후 실행"""
    env = api.Environment(cr, SUPERUSER_ID, {})
    partners = env['res.partner'].search([])
    for partner in partners:
        partner.compute_something()
```

#### 버전 간 마이그레이션 (Odoo 16 → 17)

```python
# migrations/17.0.1.0.0/pre-migration.py
def migrate(cr, version):
    """메이저 버전 업그레이드 시 수동 마이그레이션"""
    cr.execute("""
        ALTER TABLE my_table 
        RENAME COLUMN old_name TO new_name
    """)

# migrations/17.0.1.0.0/post-migration.py  
def migrate(cr, version):
    """데이터 변환"""
    cr.execute("""
        UPDATE my_table SET status = 'done' WHERE status = 'completed'
    """)
```

**결론**: Odoo는 **자체 마이그레이션 시스템**을 갖추고 있어 Alembic이 불필요합니다. 단, 복잡한 변경은 migration 폴더에 스크립트를 작성합니다.

---

## 4. Community vs Enterprise 비교

### 2.1 주요 차이점

| 항목 | Community (무료) | Enterprise (유료) |
|------|------------------|-------------------|
| **라이선스** | LGPL v3 (오픈소스) | 독점 (OEEL-1) |
| **소스 코드** | GitHub 공개 | 비공개 |
| **핵심 앱** | 제한된 세트 | 전체 스위트 |
| **기술 지원** | 커뮤니티 포럼만 | 공식 지원 포함 |
| **업그레이드** | 수동 | 포함 |
| **모바일 앱** | ❌ | ✅ Android & iOS |
| **Studio** | ❌ | ✅ 노코드 커스터마이징 |
| **회계** | 기본 기능 | 전체 기능 (OCR, 리포트, 현지화) |
| **바코드** | ❌ | ✅ |
| **PLM** | ❌ | ✅ |
| **품질 관리** | ❌ | ✅ |
| **헬프데스크** | ❌ | ✅ |
| **마케팅 자동화** | ❌ | ✅ |

### 2.2 Enterprise 가격 정책 (2025년 기준)

| 플랜 | 가격 | 특징 |
|------|------|------|
| **One App Free** | $0/월 | 단일 앱, 무제한 사용자 |
| **Standard** | ~$13.50-16.90/user/월 | 전체 앱, Odoo Online만 |
| **Custom** | ~$20.40-25.50/user/월 | 전체 앱 + Studio + 다중 회사 + 외부 API |

### 2.3 선택 가이드

**Community 선택 시**:
- 개발 학습 목적
- 기본 ERP 기능만 필요
- 커스터마이징 능력 보유
- 예산 제한

**Enterprise 선택 시**:
- 공식 지원 필요
- 고급 기능 필요 (PLM, 품질관리 등)
- 모바일 앱 필수
- Studio로 빠른 커스터마이징 원함

---

## 5. Odoo 17 vs Odoo 18 비교

> **결정**: Odoo 17 (안정 버전) 선택

### 5.1 버전별 특징

| 항목 | Odoo 17 (2023.10~) | Odoo 18 (2024.10~) |
|------|-------------------|-------------------|
| **안정성** | ✅ 안정, 검증됨 | ⚠️ 신규, 버그 가능성 |
| **커뮤니티 자료** | 풍부함 | 제한적 |
| **기업 채택** | 높음 | 도입 초기 |
| **Python** | 3.10+ | 3.10+ |
| **OWL** | 2.x | 2.x (개선됨) |

### 5.2 Odoo 18 주요 변경사항

1. **UI/UX 개선**
   - 더 현대적인 인터페이스
   - 향상된 대시보드
   - 개선된 검색 기능

2. **OWL 프레임워크 개선**
   - 더 나은 import 일관성
   - 향상된 리소스 조직
   - 최적화된 성능

3. **새로운 기능**
   - PDF 리포트 워터마크
   - 향상된 파트너 주소 관리
   - 개선된 접근 제어

4. **기술적 변경**
   - 일부 API 변경
   - 새로운 위젯 추가
   - 향상된 hook 시스템

### 5.3 버전 선택 권장

```
면접 준비 → Odoo 17 (안정성, 자료 풍부)
최신 기술 시연 → Odoo 18 (단, 회사 확인 필요)
```

**Tip**: 면접 시 회사가 사용하는 버전 확인 후, 해당 버전 중심으로 학습

---

## 6. 커스터마이징 방식

### 6.1 모델 상속 (Python)

#### 6.1.1 Classical Inheritance (새 모델 생성)
```python
class MyModel(models.Model):
    _name = 'my.model'
    _inherit = 'base.model'  # 기존 모델에서 상속
```

#### 6.1.2 Extension (기존 모델 확장) - 가장 많이 사용
```python
class ResPartner(models.Model):
    _inherit = 'res.partner'  # res.partner 확장
    
    custom_field = fields.Char('커스텀 필드')
    employee_count = fields.Integer('직원 수')
```

#### 6.1.3 Delegation (위임)
```python
class MyModel(models.Model):
    _name = 'my.model'
    _inherits = {'res.partner': 'partner_id'}
    partner_id = fields.Many2one('res.partner', required=True, ondelete='cascade')
```

### 6.2 CRUD 메서드 오버라이드

```python
class MyModel(models.Model):
    _inherit = 'existing.model'
    
    @api.model
    def create(self, vals):
        # 생성 전 로직
        if 'name' in vals:
            vals['name'] = vals['name'].upper()
        record = super().create(vals)
        # 생성 후 로직
        return record
    
    def write(self, vals):
        # 수정 전 로직
        result = super().write(vals)
        # 수정 후 로직
        return result
    
    @api.ondelete(at_uninstall=False)
    def _unlink_check(self):
        if any(rec.state != 'draft' for rec in self):
            raise UserError("초안이 아닌 레코드는 삭제할 수 없습니다")
```

### 6.3 뷰 상속 (XML)

```xml
<record id="inherited_view_form" model="ir.ui.view">
    <field name="name">my.inherited.view.form</field>
    <field name="model">res.partner</field>
    <field name="inherit_id" ref="base.view_partner_form"/>
    <field name="arch" type="xml">
        <!-- 기존 필드 뒤에 추가 -->
        <xpath expr="//field[@name='email']" position="after">
            <field name="custom_field"/>
        </xpath>
        
        <!-- 요소 교체 -->
        <xpath expr="//field[@name='phone']" position="replace">
            <field name="phone" widget="phone"/>
        </xpath>
        
        <!-- 속성 수정 -->
        <xpath expr="//field[@name='name']" position="attributes">
            <attribute name="readonly">1</attribute>
        </xpath>
    </field>
</record>
```

#### XPath Position 옵션
- `before`: 요소 앞에 삽입
- `after`: 요소 뒤에 삽입
- `inside`: 요소 내부에 삽입
- `replace`: 요소 교체
- `attributes`: 속성만 수정

### 6.4 커스터마이징 베스트 프랙티스

1. **코어 모듈 직접 수정 금지** - 항상 상속 사용
2. **의미 있는 모듈 네이밍**: `company_feature` 형식
3. **커스텀 필드 접두사**: `x_` (Studio) 또는 모듈 접두사
4. **코드 문서화**
5. **Odoo 코딩 가이드라인 준수** (PEP8)

---

## 7. 데이터 Import/Migration

### 7.1 내장 Import 도구

- **UI Import**: 설정 → Import → CSV/Excel 업로드
- 필드 매핑 지원
- 관계 데이터 처리
- 미리보기 및 검증

### 7.2 외부 API (XML-RPC / JSON-RPC)

```python
import xmlrpc.client

# 연결 설정
url = 'https://mycompany.odoo.com'
db = 'mydb'
username = 'admin'
password = 'api_key'  # 비밀번호 대신 API 키 사용

# 인증
common = xmlrpc.client.ServerProxy(f'{url}/xmlrpc/2/common')
uid = common.authenticate(db, username, password, {})

# 모델 접근
models = xmlrpc.client.ServerProxy(f'{url}/xmlrpc/2/object')

# CRUD 작업
# Search
ids = models.execute_kw(db, uid, password, 'res.partner', 'search',
    [[['is_company', '=', True]]], {'limit': 10})

# Read
records = models.execute_kw(db, uid, password, 'res.partner', 'read',
    [ids], {'fields': ['name', 'email']})

# Create
new_id = models.execute_kw(db, uid, password, 'res.partner', 'create',
    [{'name': 'New Partner', 'email': 'test@example.com'}])

# Write
models.execute_kw(db, uid, password, 'res.partner', 'write',
    [[new_id], {'phone': '123456'}])

# Delete
models.execute_kw(db, uid, password, 'res.partner', 'unlink', [[new_id]])
```

### 7.3 ORM 대량 작업

```python
# 효율적인 대량 생성
records = self.env['my.model'].create([
    {'name': 'Record 1'},
    {'name': 'Record 2'},
    {'name': 'Record 3'},
])

# 대량 수정
records.write({'state': 'done'})

# 배치 처리
for batch in self.env['my.model'].search([]).grouped(100):
    batch.process_records()
```

### 7.4 마이그레이션 전략

1. **데이터 Export/Import**: 단순 마이그레이션
2. **XML-RPC 스크립트**: 복잡한 변환
3. **마이그레이션 모듈**: `pre_init_hook`, `post_init_hook` 활용
4. **직접 SQL**: 최후의 수단 (ORM 보안 우회)

---

## 8. PM & 개발 워크플로우

### 8.1 개발 라이프사이클

```
요구사항 분석 → 개발 → 테스트 → 코드 리뷰 → 스테이징 → 프로덕션
```

### 8.2 버전 관리 베스트 프랙티스

- Git 사용, 적절한 브랜칭 전략
- 브랜치 네이밍: `feature/module-description` 또는 `fix/issue-number`
- 커스텀 모듈용 별도 저장소
- Odoo 코어와 커스텀 코드 분리

### 8.3 테스트 방식

#### Python 단위 테스트
```python
from odoo.tests import TransactionCase, tagged

@tagged('post_install', '-at_install')
class TestMyModel(TransactionCase):
    def setUp(self):
        super().setUp()
        self.partner = self.env['res.partner'].create({'name': 'Test'})
    
    def test_some_feature(self):
        result = self.partner.some_method()
        self.assertEqual(result, expected_value)
```

#### 테스트 실행
```bash
# 모듈 전체 테스트
odoo-bin -i my_module --test-enable

# 특정 테스트
odoo-bin --test-tags /my_module

# 특정 테스트 클래스
odoo-bin --test-tags /my_module:TestMyModel
```

### 8.4 배포 전략

| 방식 | 설명 | 장점 |
|------|------|------|
| **Odoo.sh** | Odoo 공식 PaaS | Git 기반 배포, CI/CD 자동화 |
| **On-Premise** | 직접 서버 운영 | 완전한 통제권 |
| **Docker** | 컨테이너 기반 | 이식성, 확장성 |

---

## 9. 개발 환경 선택

### 9.1 환경 비교

| 환경 | 장점 | 단점 |
|------|------|------|
| **로컬 Python** | 완전한 통제, 디버깅 용이, Git 관리 | 초기 설정 복잡 |
| **Odoo.sh** | CI/CD 자동화, 스테이징/프로덕션 분리, 백업 | 무료 플랜 제한적 |
| **Docker** | 이식성, 환경 일관성, 팀 협업 용이 | 추가 학습 필요 |

### 9.2 Odoo.sh 무료 플랜 장점

1. **Git 통합**: Push하면 자동 배포
2. **CI/CD 파이프라인**: 자동 테스트 및 배포
3. **스테이징 환경**: 프로덕션 배포 전 테스트
4. **백업**: 자동 백업
5. **모니터링**: 성능 모니터링 대시보드
6. **면접 시연**: URL로 바로 시연 가능

**무료 플랜 제한**:
- 제한된 컴퓨팅 리소스
- 제한된 사용자 수
- 일부 기능 제한

### 9.3 권장 접근법

```
Step 1: 로컬 Python 환경 구축 (개발 & 학습)
Step 2: Git에 커밋 및 관리
Step 3: Odoo.sh 연동 (면접 시연용 - 선택)
Step 4: Docker 컨테이너화 (이후 배포/협업용)
```

### 9.4 로컬 환경 구축 (Windows)

```powershell
# 1. Python 3.10+ 설치 확인
python --version

# 2. PostgreSQL 설치 및 사용자 생성
# PostgreSQL 설치 후:
createuser -U postgres -s odoo

# 3. Odoo 소스 클론
git clone https://github.com/odoo/odoo.git --depth 1 --branch 17.0

# 4. 가상환경 생성
python -m venv venv
.\venv\Scripts\Activate.ps1

# 5. 의존성 설치
pip install -r odoo/requirements.txt

# 6. Odoo 실행
python odoo/odoo-bin --addons-path=odoo/addons,custom_addons -d mydb
```

---

## 10. 학습 로드맵

### 10.1 Phase 1: 데모 수준 (1-2주)

#### HR 모듈 (첫 번째)
- [ ] `hr` 모듈 분석 및 이해
- [ ] 직원 정보에 커스텀 필드 추가
- [ ] 간단한 뷰 상속 실습
- [ ] 기본 CRUD 테스트

#### 목표 결과물
```
custom_hr/
├── __manifest__.py
├── models/
│   └── hr_employee.py    # 직원 모델 확장
├── views/
│   └── hr_employee_views.xml
└── security/
    └── ir.model.access.csv
```

### 10.2 Phase 2: 재고관리 확장 (2-3주)

#### Stock/Inventory 모듈
- [ ] `stock` 모듈 구조 분석
- [ ] 재고 이동 커스터마이징
- [ ] 창고 관리 확장
- [ ] 바코드 스캔 연동 (선택)

#### 목표 결과물
```
custom_stock/
├── __manifest__.py
├── models/
│   ├── stock_move.py
│   └── stock_warehouse.py
├── views/
│   └── stock_views.xml
└── reports/
    └── stock_report.xml
```

### 10.3 Phase 3: 영업 모듈 (3-4주)

#### Sales 모듈
- [ ] `sale` 모듈 분석
- [ ] 견적서 템플릿 커스터마이징
- [ ] 할인 정책 구현
- [ ] 리포트 생성

#### 목표 결과물
```
custom_sale/
├── __manifest__.py
├── models/
│   ├── sale_order.py
│   └── sale_order_line.py
├── views/
│   └── sale_views.xml
├── reports/
│   └── sale_report.xml
└── wizards/
    └── sale_discount_wizard.py
```

### 10.4 Phase 4: 실무 적용 마이그레이션 (4주+)

- [ ] 모듈 간 연동 (HR ↔ Stock ↔ Sale)
- [ ] API 개발 (외부 시스템 연동)
- [ ] 성능 최적화
- [ ] 단위 테스트 작성
- [ ] 문서화

---

## 면접 준비 포인트

### 기술적 깊이
- `_inherit` vs `_inherits` 차이 설명
- Computed fields와 의존성 동작 방식
- Recordset 개념과 캐싱 메커니즘
- 보안 레이어 (ir.model.access, ir.rules)

### 실무 경험
- 복잡한 커스터마이징 사례
- 버전 간 데이터 마이그레이션 방법
- Odoo 이슈 디버깅 접근법
- 코드 품질 보장 방법

### 베스트 프랙티스
- `@api.model` vs `@api.depends` 사용 시점
- `sudo()` 적절한 사용과 보안 영향
- 성능 고려사항 (prefetching, bulk operations)
- 모듈 의존성 관리

### 최신 지식
- OWL 프레임워크 (Odoo 17/18)
- 최신 버전의 새 기능
- Odoo.sh CI/CD 파이프라인 이해

---

## 참고 자료

- [Odoo 공식 문서](https://www.odoo.com/documentation/17.0/)
- [Odoo GitHub](https://github.com/odoo/odoo)
- [Odoo 튜토리얼](https://www.odoo.com/slides)
- [Odoo 포럼](https://www.odoo.com/forum/help-1)
- [Odoo.sh 문서](https://www.odoo.sh/documentation)

---

> **다음 단계**: 로컬 Python 환경 구축 → 첫 번째 커스텀 HR 모듈 생성
