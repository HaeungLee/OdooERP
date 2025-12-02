# Odoo ERP 개발 분석 문서

> **작성일**: 2024-12-02  
> **목적**: Odoo 개발/커스터마이징 직무 면접 준비 및 학습  
> **실습 방향**: HR → 재고관리 → 영업  

---
## 목차

1. [Odoo 아키텍처 & 사용법](#1-odoo-아키텍처--사용법)
2. [Community vs Enterprise 비교](#2-community-vs-enterprise-비교)
3. [Odoo 17 vs Odoo 18 비교](#3-odoo-17-vs-odoo-18-비교)
4. [커스터마이징 방식](#4-커스터마이징-방식)
5. [데이터 Import/Migration](#5-데이터-importmigration)
6. [PM & 개발 워크플로우](#6-pm--개발-워크플로우)
7. [개발 환경 선택](#7-개발-환경-선택)
8. [학습 로드맵](#8-학습-로드맵)

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

## 2. Community vs Enterprise 비교

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

## 3. Odoo 17 vs Odoo 18 비교

> **결정**: Odoo 17 (안정 버전) 선택

### 3.1 버전별 특징

| 항목 | Odoo 17 (2023.10~) | Odoo 18 (2024.10~) |
|------|-------------------|-------------------|
| **안정성** | ✅ 안정, 검증됨 | ⚠️ 신규, 버그 가능성 |
| **커뮤니티 자료** | 풍부함 | 제한적 |
| **기업 채택** | 높음 | 도입 초기 |
| **Python** | 3.10+ | 3.10+ |
| **OWL** | 2.x | 2.x (개선됨) |

### 3.2 Odoo 18 주요 변경사항

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

### 3.3 버전 선택 권장

```
면접 준비 → Odoo 17 (안정성, 자료 풍부)
최신 기술 시연 → Odoo 18 (단, 회사 확인 필요)
```

**Tip**: 면접 시 회사가 사용하는 버전 확인 후, 해당 버전 중심으로 학습

---

## 4. 커스터마이징 방식

### 4.1 모델 상속 (Python)

#### 4.1.1 Classical Inheritance (새 모델 생성)
```python
class MyModel(models.Model):
    _name = 'my.model'
    _inherit = 'base.model'  # 기존 모델에서 상속
```

#### 4.1.2 Extension (기존 모델 확장) - 가장 많이 사용
```python
class ResPartner(models.Model):
    _inherit = 'res.partner'  # res.partner 확장
    
    custom_field = fields.Char('커스텀 필드')
    employee_count = fields.Integer('직원 수')
```

#### 4.1.3 Delegation (위임)
```python
class MyModel(models.Model):
    _name = 'my.model'
    _inherits = {'res.partner': 'partner_id'}
    partner_id = fields.Many2one('res.partner', required=True, ondelete='cascade')
```

### 4.2 CRUD 메서드 오버라이드

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

### 4.3 뷰 상속 (XML)

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

### 4.4 커스터마이징 베스트 프랙티스

1. **코어 모듈 직접 수정 금지** - 항상 상속 사용
2. **의미 있는 모듈 네이밍**: `company_feature` 형식
3. **커스텀 필드 접두사**: `x_` (Studio) 또는 모듈 접두사
4. **코드 문서화**
5. **Odoo 코딩 가이드라인 준수** (PEP8)

---

## 5. 데이터 Import/Migration

### 5.1 내장 Import 도구

- **UI Import**: 설정 → Import → CSV/Excel 업로드
- 필드 매핑 지원
- 관계 데이터 처리
- 미리보기 및 검증

### 5.2 외부 API (XML-RPC / JSON-RPC)

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

### 5.3 ORM 대량 작업

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

### 5.4 마이그레이션 전략

1. **데이터 Export/Import**: 단순 마이그레이션
2. **XML-RPC 스크립트**: 복잡한 변환
3. **마이그레이션 모듈**: `pre_init_hook`, `post_init_hook` 활용
4. **직접 SQL**: 최후의 수단 (ORM 보안 우회)

---

## 6. PM & 개발 워크플로우

### 6.1 개발 라이프사이클

```
요구사항 분석 → 개발 → 테스트 → 코드 리뷰 → 스테이징 → 프로덕션
```

### 6.2 버전 관리 베스트 프랙티스

- Git 사용, 적절한 브랜칭 전략
- 브랜치 네이밍: `feature/module-description` 또는 `fix/issue-number`
- 커스텀 모듈용 별도 저장소
- Odoo 코어와 커스텀 코드 분리

### 6.3 테스트 방식

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

### 6.4 배포 전략

| 방식 | 설명 | 장점 |
|------|------|------|
| **Odoo.sh** | Odoo 공식 PaaS | Git 기반 배포, CI/CD 자동화 |
| **On-Premise** | 직접 서버 운영 | 완전한 통제권 |
| **Docker** | 컨테이너 기반 | 이식성, 확장성 |

---

## 7. 개발 환경 선택

### 7.1 환경 비교

| 환경 | 장점 | 단점 |
|------|------|------|
| **로컬 Python** | 완전한 통제, 디버깅 용이, Git 관리 | 초기 설정 복잡 |
| **Odoo.sh** | CI/CD 자동화, 스테이징/프로덕션 분리, 백업 | 무료 플랜 제한적 |
| **Docker** | 이식성, 환경 일관성, 팀 협업 용이 | 추가 학습 필요 |

### 7.2 Odoo.sh 무료 플랜 장점

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

### 7.3 권장 접근법

```
Step 1: 로컬 Python 환경 구축 (개발 & 학습)
Step 2: Git에 커밋 및 관리
Step 3: Odoo.sh 연동 (면접 시연용 - 선택)
Step 4: Docker 컨테이너화 (이후 배포/협업용)
```

### 7.4 로컬 환경 구축 (Windows)

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

## 8. 학습 로드맵

### 8.1 Phase 1: 데모 수준 (1-2주)

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

### 8.2 Phase 2: 재고관리 확장 (2-3주)

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

### 8.3 Phase 3: 영업 모듈 (3-4주)

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

### 8.4 Phase 4: 실무 적용 마이그레이션 (4주+)

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
