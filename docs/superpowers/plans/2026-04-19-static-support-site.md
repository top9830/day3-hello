# 정적 고객지원 웹사이트 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 기존 고객이 공지 확인과 카테고리형 FAQ 탐색으로 스스로 문제를 해결하고, 필요 시 채팅 문의로 빠르게 전환할 수 있는 3페이지 정적 사이트를 구축한다.

**Architecture:** 순수 정적 HTML/CSS와 최소 JavaScript(필요 시)로 `홈/FAQ/문의` 3페이지를 구성한다. 자동 검증은 Python `unittest` 기반으로 페이지 구조, 콘텐츠 존재, 링크 무결성, 접근성 최소 기준(viewport, focus-visible)을 확인한다. 운영팀 수시 업데이트 요구를 반영해 콘텐츠는 각 페이지 내 명확한 섹션 단위로 배치한다.

**Tech Stack:** HTML5, CSS3, Vanilla JavaScript(optional), Python 3 `unittest`

---

## 파일 구조

- Create: `index.html` — 공지 배너, 후기 하이라이트, FAQ 이동 CTA가 있는 홈 화면
- Create: `faq.html` — 카테고리 중심 FAQ 탐색(시작하기/요금/정책/문제해결)
- Create: `contact.html` — 채팅 링크 중심 문의 페이지
- Create: `assets/styles.css` — 공통 레이아웃, 반응형, 포커스 스타일
- Create: `tests/__init__.py` — 테스트 패키지 인식
- Create/Modify: `tests/test_site.py` — 구조/콘텐츠/링크/접근성 자동 검증

---

### Task 1: 기본 페이지 골격과 전역 네비게이션 구축

**Files:**
- Create: `tests/__init__.py`
- Create: `tests/test_site.py`
- Create: `index.html`
- Create: `faq.html`
- Create: `contact.html`
- Create: `assets/styles.css`

- [ ] **Step 1: 실패하는 구조 테스트 작성**

```python
# tests/test_site.py
import pathlib
import unittest

ROOT = pathlib.Path(__file__).resolve().parents[1]


class SiteStructureTest(unittest.TestCase):
    def read_page(self, name: str) -> str:
        return (ROOT / name).read_text(encoding="utf-8")

    def test_required_files_exist(self):
        for rel_path in ["index.html", "faq.html", "contact.html", "assets/styles.css"]:
            self.assertTrue((ROOT / rel_path).exists(), f"Missing {rel_path}")

    def test_global_navigation_links_exist_on_all_pages(self):
        expected_links = ['href="index.html"', 'href="faq.html"', 'href="contact.html"']
        for page in ["index.html", "faq.html", "contact.html"]:
            html = self.read_page(page)
            for link in expected_links:
                self.assertIn(link, html, f"{page} missing {link}")
```

- [ ] **Step 2: 테스트 실행으로 실패 확인**

Run: `python -m unittest tests.test_site.SiteStructureTest.test_required_files_exist`
Expected: FAIL with `Missing index.html`

- [ ] **Step 3: 최소 구현으로 테스트 통과시키기**

```html
<!-- index.html -->
<!doctype html>
<html lang="ko">
  <head>
    <meta charset="utf-8" />
    <title>고객지원 홈</title>
    <link rel="stylesheet" href="assets/styles.css" />
  </head>
  <body>
    <header>
      <nav>
        <a href="index.html">홈</a>
        <a href="faq.html">FAQ</a>
        <a href="contact.html">문의</a>
      </nav>
    </header>
    <main>
      <h1>고객지원 센터</h1>
    </main>
  </body>
</html>
```

```html
<!-- faq.html -->
<!doctype html>
<html lang="ko">
  <head>
    <meta charset="utf-8" />
    <title>FAQ</title>
    <link rel="stylesheet" href="assets/styles.css" />
  </head>
  <body>
    <header>
      <nav>
        <a href="index.html">홈</a>
        <a href="faq.html">FAQ</a>
        <a href="contact.html">문의</a>
      </nav>
    </header>
    <main>
      <h1>자주 묻는 질문</h1>
    </main>
  </body>
</html>
```

```html
<!-- contact.html -->
<!doctype html>
<html lang="ko">
  <head>
    <meta charset="utf-8" />
    <title>문의</title>
    <link rel="stylesheet" href="assets/styles.css" />
  </head>
  <body>
    <header>
      <nav>
        <a href="index.html">홈</a>
        <a href="faq.html">FAQ</a>
        <a href="contact.html">문의</a>
      </nav>
    </header>
    <main>
      <h1>문의하기</h1>
    </main>
  </body>
</html>
```

```css
/* assets/styles.css */
* {
  box-sizing: border-box;
}

body {
  margin: 0;
  font-family: "Noto Sans KR", Arial, sans-serif;
  color: #1f2937;
  background: #f9fafb;
}

nav {
  display: flex;
  gap: 16px;
  padding: 16px 24px;
  background: #ffffff;
  border-bottom: 1px solid #e5e7eb;
}

main {
  max-width: 960px;
  margin: 0 auto;
  padding: 32px 24px;
}
```

```text
# tests/__init__.py
```

- [ ] **Step 4: 구조 테스트 재실행**

Run: `python -m unittest tests.test_site.SiteStructureTest.test_required_files_exist tests.test_site.SiteStructureTest.test_global_navigation_links_exist_on_all_pages`
Expected: PASS

- [ ] **Step 5: 커밋**

```bash
git add tests/__init__.py tests/test_site.py index.html faq.html contact.html assets/styles.css
git commit -m "feat: scaffold static support site pages and global navigation"
```

---

### Task 2: 홈 페이지(공지/후기/FAQ CTA) 구현

**Files:**
- Modify: `tests/test_site.py`
- Modify: `index.html`

- [ ] **Step 1: 홈 요구사항 실패 테스트 추가**

```python
# tests/test_site.py (SiteStructureTest에 추가)
    def test_home_has_notice_banner_testimonial_and_faq_cta(self):
        html = self.read_page("index.html")

        banner_pos = html.find('id="notice-banner"')
        testimonial_pos = html.find('id="testimonial-highlight"')
        cta_pos = html.find('id="faq-cta"')

        self.assertNotEqual(banner_pos, -1, "공지 배너가 없습니다")
        self.assertNotEqual(testimonial_pos, -1, "고객 후기 하이라이트가 없습니다")
        self.assertNotEqual(cta_pos, -1, "FAQ CTA가 없습니다")
        self.assertTrue(banner_pos < testimonial_pos < cta_pos, "홈 정보 우선순위가 맞지 않습니다")

    def test_home_notice_banner_text_exists(self):
        html = self.read_page("index.html")
        self.assertIn("공지/업데이트", html)
```

- [ ] **Step 2: 테스트 실패 확인**

Run: `python -m unittest tests.test_site.SiteStructureTest.test_home_has_notice_banner_testimonial_and_faq_cta tests.test_site.SiteStructureTest.test_home_notice_banner_text_exists`
Expected: FAIL with missing `notice-banner` / `공지/업데이트`

- [ ] **Step 3: 홈 콘텐츠 구현**

```html
<!-- index.html main 내부를 아래로 교체 -->
<main>
  <h1>고객지원 센터</h1>

  <section id="notice-banner" aria-label="공지 배너">
    <h2>공지/업데이트</h2>
    <p>이번 주 정책 변경 사항과 점검 일정을 먼저 확인해 주세요.</p>
  </section>

  <section id="core-guidance" aria-label="핵심 안내">
    <h2>먼저 FAQ에서 빠르게 답을 찾아보세요</h2>
    <p>카테고리별 질문 정리로 문의 전 자체 해결을 돕습니다.</p>
  </section>

  <section id="testimonial-highlight" aria-label="고객 후기">
    <h2>고객 후기</h2>
    <blockquote>
      "문제해결 카테고리에서 바로 답을 찾아 문의 없이 해결했습니다."
    </blockquote>
    <p>- 기존 고객 A</p>
  </section>

  <p>
    <a id="faq-cta" href="faq.html">FAQ 바로가기</a>
  </p>
</main>
```

- [ ] **Step 4: 테스트 재실행**

Run: `python -m unittest tests.test_site.SiteStructureTest.test_home_has_notice_banner_testimonial_and_faq_cta tests.test_site.SiteStructureTest.test_home_notice_banner_text_exists`
Expected: PASS

- [ ] **Step 5: 커밋**

```bash
git add tests/test_site.py index.html
git commit -m "feat: add prioritized home sections with notice banner and faq cta"
```

---

### Task 3: FAQ 카테고리 탐색 구조 구현

**Files:**
- Modify: `tests/test_site.py`
- Modify: `faq.html`

- [ ] **Step 1: FAQ 카테고리 실패 테스트 추가**

```python
# tests/test_site.py (SiteStructureTest에 추가)
    def test_faq_has_category_navigation(self):
        html = self.read_page("faq.html")
        for token in [
            'href="#getting-started"',
            'href="#pricing"',
            'href="#policy"',
            'href="#troubleshooting"',
        ]:
            self.assertIn(token, html)

    def test_faq_has_required_category_sections(self):
        html = self.read_page("faq.html")
        for section_id in ["getting-started", "pricing", "policy", "troubleshooting"]:
            self.assertIn(f'id="{section_id}"', html)
```

- [ ] **Step 2: 테스트 실패 확인**

Run: `python -m unittest tests.test_site.SiteStructureTest.test_faq_has_category_navigation tests.test_site.SiteStructureTest.test_faq_has_required_category_sections`
Expected: FAIL with missing category anchors/sections

- [ ] **Step 3: FAQ 페이지 구현**

```html
<!-- faq.html main 내부를 아래로 교체 -->
<main>
  <h1>자주 묻는 질문</h1>

  <nav aria-label="FAQ 카테고리">
    <a href="#getting-started">시작하기</a>
    <a href="#pricing">요금</a>
    <a href="#policy">정책</a>
    <a href="#troubleshooting">문제해결</a>
  </nav>

  <section id="getting-started">
    <h2>시작하기</h2>
    <p>서비스 첫 이용 절차와 기본 설정 안내입니다.</p>
  </section>

  <section id="pricing">
    <h2>요금</h2>
    <p>요금제별 제공 기능과 청구 주기를 안내합니다.</p>
  </section>

  <section id="policy">
    <h2>정책</h2>
    <p>환불, 약관, 개인정보 처리 정책을 확인할 수 있습니다.</p>
  </section>

  <section id="troubleshooting">
    <h2>문제해결</h2>
    <p>자주 발생하는 오류 상황과 해결 방법을 제공합니다.</p>
  </section>
</main>
```

- [ ] **Step 4: 테스트 재실행**

Run: `python -m unittest tests.test_site.SiteStructureTest.test_faq_has_category_navigation tests.test_site.SiteStructureTest.test_faq_has_required_category_sections`
Expected: PASS

- [ ] **Step 5: 커밋**

```bash
git add tests/test_site.py faq.html
git commit -m "feat: implement category-driven faq navigation and sections"
```

---

### Task 4: 문의 페이지 채팅 링크와 FAQ→문의 전환 동선 구현

**Files:**
- Modify: `tests/test_site.py`
- Modify: `faq.html`
- Modify: `contact.html`

- [ ] **Step 1: 문의 동선 실패 테스트 추가**

```python
# tests/test_site.py (SiteStructureTest에 추가)
    def test_contact_has_primary_chat_link(self):
        html = self.read_page("contact.html")
        self.assertIn('id="chat-link"', html)
        self.assertIn('href="https://example.com/chat"', html)
        self.assertIn('target="_blank"', html)
        self.assertIn('rel="noopener noreferrer"', html)

    def test_faq_has_contact_cta_for_unresolved_questions(self):
        html = self.read_page("faq.html")
        self.assertIn('href="contact.html#chat-link"', html)
```

- [ ] **Step 2: 테스트 실패 확인**

Run: `python -m unittest tests.test_site.SiteStructureTest.test_contact_has_primary_chat_link tests.test_site.SiteStructureTest.test_faq_has_contact_cta_for_unresolved_questions`
Expected: FAIL with missing chat link / CTA

- [ ] **Step 3: 문의 동선 구현**

```html
<!-- contact.html main 내부를 아래로 교체 -->
<main>
  <h1>문의하기</h1>
  <p>FAQ에서 해결되지 않았다면 채팅으로 바로 문의해 주세요.</p>
  <p>
    <a id="chat-link" href="https://example.com/chat" target="_blank" rel="noopener noreferrer">
      채팅으로 문의하기
    </a>
  </p>
</main>
```

```html
<!-- faq.html main 맨 아래에 추가 -->
<section aria-label="추가 문의 안내">
  <h2>원하는 답을 찾지 못하셨나요?</h2>
  <p><a href="contact.html#chat-link">채팅 문의로 바로 이동</a></p>
</section>
```

- [ ] **Step 4: 테스트 재실행**

Run: `python -m unittest tests.test_site.SiteStructureTest.test_contact_has_primary_chat_link tests.test_site.SiteStructureTest.test_faq_has_contact_cta_for_unresolved_questions`
Expected: PASS

- [ ] **Step 5: 커밋**

```bash
git add tests/test_site.py faq.html contact.html
git commit -m "feat: add chat-first contact flow from faq"
```

---

### Task 5: 반응형/접근성 최소 기준과 링크 무결성 검증 완성

**Files:**
- Modify: `tests/test_site.py`
- Modify: `index.html`
- Modify: `faq.html`
- Modify: `contact.html`
- Modify: `assets/styles.css`

- [ ] **Step 1: 품질 기준 실패 테스트 추가**

```python
# tests/test_site.py 상단 import 추가
import re

# tests/test_site.py (SiteStructureTest에 추가)
    def test_all_pages_have_viewport_meta(self):
        token = '<meta name="viewport" content="width=device-width, initial-scale=1" />'
        for page in ["index.html", "faq.html", "contact.html"]:
            html = self.read_page(page)
            self.assertIn(token, html, f"{page} missing viewport meta")

    def test_each_page_has_single_h1(self):
        for page in ["index.html", "faq.html", "contact.html"]:
            html = self.read_page(page)
            self.assertEqual(html.count("<h1"), 1, f"{page} must have exactly one h1")

    def test_styles_include_focus_visible(self):
        css = (ROOT / "assets/styles.css").read_text(encoding="utf-8")
        self.assertIn(":focus-visible", css)

    def test_local_links_are_not_broken(self):
        pages = ["index.html", "faq.html", "contact.html"]
        for page in pages:
            html = self.read_page(page)
            hrefs = re.findall(r'href="([^"]+)"', html)
            for href in hrefs:
                if href.startswith(("http://", "https://", "mailto:", "tel:")):
                    continue
                if href.startswith("#"):
                    anchor = href[1:]
                    self.assertIn(f'id="{anchor}"', html, f"{page} missing anchor #{anchor}")
                    continue

                if "#" in href:
                    file_name, anchor = href.split("#", 1)
                else:
                    file_name, anchor = href, ""

                target_file = ROOT / file_name
                self.assertTrue(target_file.exists(), f"{page} links to missing file {file_name}")

                if anchor:
                    target_html = target_file.read_text(encoding="utf-8")
                    self.assertIn(f'id="{anchor}"', target_html, f"{file_name} missing anchor #{anchor}")
```

- [ ] **Step 2: 테스트 실패 확인**

Run: `python -m unittest tests.test_site.SiteStructureTest.test_all_pages_have_viewport_meta tests.test_site.SiteStructureTest.test_styles_include_focus_visible`
Expected: FAIL with missing viewport meta / `:focus-visible`

- [ ] **Step 3: 접근성/반응형 구현**

```html
<!-- index.html, faq.html, contact.html의 <head>에 동일하게 추가 -->
<meta name="viewport" content="width=device-width, initial-scale=1" />
```

```css
/* assets/styles.css 하단에 추가 */
a {
  color: #1d4ed8;
}

a:focus-visible {
  outline: 3px solid #2563eb;
  outline-offset: 2px;
  border-radius: 4px;
}

#notice-banner {
  background: #eff6ff;
  border: 1px solid #bfdbfe;
  padding: 16px;
  border-radius: 8px;
}

@media (max-width: 768px) {
  nav {
    flex-wrap: wrap;
    gap: 12px;
    padding: 12px 16px;
  }

  main {
    padding: 24px 16px;
  }
}
```

- [ ] **Step 4: 전체 테스트 실행**

Run: `python -m unittest discover -s tests -p "test_site.py"`
Expected: PASS

- [ ] **Step 5: 수동 골든패스 점검 후 커밋**

Run: `python -m http.server 8080`
Expected: `Serving HTTP on ... port 8080`

Manual check:
1. `http://localhost:8080/index.html`에서 공지 배너 확인
2. FAQ CTA로 `faq.html` 이동
3. 카테고리 4종 탐색
4. FAQ 하단 CTA로 `contact.html#chat-link` 이동
5. 채팅 링크 클릭 동작 확인

```bash
git add tests/test_site.py index.html faq.html contact.html assets/styles.css
git commit -m "feat: finalize responsive accessible support site with link integrity tests"
```

---

## 최종 검증 명령어(릴리스 전)

- 자동 검증: `python -m unittest discover -s tests -p "test_site.py"`
- 수동 검증: `python -m http.server 8080` 후 홈→FAQ→문의 골든패스 점검

---

## Self-Review (완료)

1. **Spec coverage:**
   - 공지 최상단/후기/FAQ CTA: Task 2
   - 카테고리 중심 FAQ: Task 3
   - 채팅 링크 중심 문의: Task 4
   - 반응형/접근성/링크검증/골든패스: Task 5

2. **Placeholder scan:**
   - `TBD`, `TODO`, 모호한 "적절히" 표현 없음

3. **Type consistency:**
   - 파일명(`index.html`, `faq.html`, `contact.html`)과 anchor id(`chat-link`, `getting-started`, `pricing`, `policy`, `troubleshooting`) 전 태스크 일관
