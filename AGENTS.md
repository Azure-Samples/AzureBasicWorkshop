# 워크샵 에이전트 작업 지침

## 리포 규칙

- 이 저장소는 A형 포털 워크샵이다. [학습 경로](README.md#학습-경로)와 기존 실습 절차·명령을 보존한다. 노트북이나 devcontainer를 임의로 추가하지 않는다.
- `main`에 직접 커밋하지 않는다. 별도 로컬 브랜치에서 작업하고 기존 미커밋 변경을 보존한다. 원격 push·공개 전환·클라우드 배포·삭제는 자동으로 수행하지 않는다.
- 실제 자격 증명이나 개인 리소스 식별자를 커밋하지 않는다. 노출을 발견하면 값을 재출력하지 말고 파일·위치·성격만 보고한 뒤 중단한다.
- 결정 시트와 검사 원본 로그는 Git에서 제외한 `local-only`에만 보관한다. 공유할 때는 민감한 값을 제거한다.
- 랩 경로는 `NN-kebab-name`, 랩 전용 자산은 해당 랩의 `assets`를 사용한다. 여러 랩이 실제로 참조하는 자산만 `shared-assets`로 옮긴다. 개명과 링크 변경을 함께 수행한다.
- 기존 [MIT 라이선스](LICENSE.md)와 저작권 고지를 유지한다. 문서·이미지의 CC BY-SA 4.0 전환은 필요한 동의가 확인되기 전까지 보류한다.

## 콘텐츠·노트북 스타일 가이드

- 안내 문서는 한국어로 작성하되 기존 실습 코드의 동작은 변경하지 않는다. 각 랩은 현재의 단일 문서형을 유지한다.
- 루트에는 전체 frontmatter, 랩에는 `title`·`duration_minutes`·`last_updated`를 둔다. 문서 수정일과 E2E 검증일을 구분한다.
- 랩 섹션은 개요와 학습 목표 → 사전 요구사항 → 소요 시간 → 실습 단계 → 검증 → 정리 → 트러블슈팅 → 이전·다음 순서로 둔다. H1은 하나만 사용한다.
- 원문 근거가 있는 내용은 재배치해 사용한다. 시간·비용·권한·삭제 범위·실행 결과가 미확정이면 TODO로 남긴다. 실습 절차를 임의로 창작·삭제·분할하지 않는다.
- 이미지에는 화면이나 단계의 목적을 설명하는 alt-text를 작성한다. 이미지 교체·삭제는 저자 확인을 받는다.
- 향후 보조 노트북을 추가하더라도 코드 셀의 `outputs`는 빈 배열, `execution_count`는 null로 커밋한다.

## 검증 하네스

저장소 루트에서 Git Bash 또는 Bash 기반 환경으로 실행한다. gitleaks 8.19 이상, Python 3.10 이상과 PyYAML이 필요하다. 설치·업데이트는 승인받는다. 아래는 표준화 변환 지시문의 검사 9종 원문이다.

각 명령의 종료 상태와 전체 출력을 확인한다. 특히 파이프라인 뒤의 `PASS` 메시지는 앞선 오류나 검출을 무효화하지 않는다. 파일 접근 오류·도구 누락·검출이 있으면 통과로 표시하지 말고 중단한다. 로컬 검사 기록도 검색 대상이므로 실제 민감한 값을 기록에 복사하지 않는다.

```bash
# (1) 시크릿 스캔 — 이력 포함 (gitleaks >= 8.19 신문법)
gitleaks git . --log-opts="--all" || { echo "FAIL: secrets"; exit 1; }

# (2) 개인 리소스 식별자 스캔 — 문서·코드·설정
grep -rEn --include='*.md' --include='*.py' --include='*.ipynb' --include='*.sh' --include='*.bicep' --include='*.json' \
  'https?://[a-z0-9-]+\.(openai\.azure\.com|cognitiveservices\.azure\.com|services\.ai\.azure\.com|azure-api\.net)' . \
  | grep -v '<your\|example\|placeholder' && echo "FAIL: personal endpoints" || echo "PASS"

# (3) 노트북 출력 클리어 확인
python3 - <<'EOF'
import json, glob, sys
bad = [f for f in glob.glob('**/*.ipynb', recursive=True)
       if any(c.get('outputs') or c.get('execution_count') is not None
              for c in json.load(open(f))['cells'] if c['cell_type']=='code')]
print("FAIL: outputs in", bad) if bad else print("PASS: notebooks clean")
sys.exit(1 if bad else 0)
EOF

# (4) 공백·비ASCII 경로 잔존 확인
find . -path ./.git -prune -o -print | grep -P '[ ]|[^\x00-\x7F]' \
  && echo "FAIL: non-standard paths" || echo "PASS"

# (5) frontmatter 스키마 검증
python3 - <<'EOF'
import re, sys, yaml
txt = open('README.md', encoding='utf-8').read()
m = re.match(r'^---\n(.*?)\n---', txt, re.S)
assert m, "FAIL: no frontmatter"
fm = yaml.safe_load(m.group(1))
required = ['type','title','description','level','authors','contacts','duration_minutes',
            'tags','language','execution','status','source','last_updated']
missing = [k for k in required if k not in fm]
if 'validated_on' not in fm: missing.append('validated_on(키 필수, 값은 공란 허용)')
print("FAIL: missing", missing) if missing else print("PASS: frontmatter")
sys.exit(1 if missing else 0)
EOF

# (6) 깨진 상대 링크·이미지 참조 검사 (fenced/inline 코드 제외, <img src> 포함)
python3 - <<'EOF'
import re, glob, os, sys, urllib.parse
bad = []
for f in glob.glob('**/*.md', recursive=True):
    t = open(f, encoding='utf-8').read()
    t = re.sub(r'```.*?```', '', t, flags=re.S)   # fenced code block 제외 (SQL 등 오인 방지)
    t = re.sub(r'`[^`\n]*`', '', t)               # inline code 제외
    base = os.path.dirname(f)
    links = re.findall(r'\]\(([^)#\s]+?)(?:#[^)]*)?\)', t)
    links += re.findall(r'<img[^>]+src=["\']([^"\']+)["\']', t)
    for link in links:
        if link.startswith(('http', 'mailto:', 'data:')): continue
        p = os.path.normpath(os.path.join(base, urllib.parse.unquote(link.strip())))
        if not os.path.exists(p): bad.append(f"{f} -> {link}")
print("FAIL:\n" + "\n".join(bad)) if bad else print("PASS: links")
sys.exit(1 if bad else 0)
EOF

# (7) 랩 README 축약 frontmatter 검사 (title·duration_minutes·last_updated 키 존재)
python3 - <<'EOF'
import re, glob, os, sys, yaml
bad = []
for f in glob.glob('**/README.md', recursive=True):
    d = os.path.basename(os.path.dirname(f))
    if not re.match(r'^(\d{2}-|lab\d)', d): continue  # lab\d은 전환기 하위호환 — 표준은 NN-
    m = re.match(r'^---\n(.*?)\n---', open(f, encoding='utf-8').read(), re.S)
    if not m: bad.append(f + ' -> frontmatter 없음'); continue
    try: fm = yaml.safe_load(m.group(1)) or {}
    except yaml.YAMLError as e: bad.append(f + ' -> YAML 파싱 오류(따옴표 누락 등)'); continue
    miss = [k for k in ('title','duration_minutes','last_updated') if k not in fm]
    if miss: bad.append(f + ' -> 키 누락: ' + ','.join(miss))
print("FAIL:\n" + "\n".join(bad)) if bad else print("PASS: lab frontmatter")
sys.exit(1 if bad else 0)
EOF

# (8) 이미지 alt-text 검사 (빈 alt 금지 — markdown ![]와 HTML <img>)
python3 - <<'EOF'
import re, glob, sys
bad = []
for f in glob.glob('**/*.md', recursive=True):
    t = re.sub(r'```.*?```', '', open(f, encoding='utf-8').read(), flags=re.S)
    if re.search(r'!\[\s*\]\(', t): bad.append(f + ' -> 빈 alt (markdown)')
    for img in re.findall(r'<img[^>]*>', t):
        m = re.search(r'alt=["\']([^"\']*)["\']', img)
        if not m or not m.group(1).strip(): bad.append(f + ' -> 빈 alt (<img>)')
print("FAIL:\n" + "\n".join(bad)) if bad else print("PASS: alt-text")
sys.exit(1 if bad else 0)
EOF

# (9) 랩 전용 자산 교차 참조 검사 (다른 랩의 assets/ 참조 = shared-assets 이동 대상)
python3 - <<'EOF'
import re, glob, os, sys, urllib.parse
bad = []
for f in glob.glob('**/*.md', recursive=True):
    t = re.sub(r'```.*?```', '', open(f, encoding='utf-8').read(), flags=re.S)
    base = os.path.dirname(f).replace(os.sep, '/')
    links = re.findall(r'\]\(([^)#\s]+?)\)', t) + re.findall(r'<img[^>]+src=["\']([^"\']+)["\']', t)
    for l in links:
        if l.startswith(('http','mailto:','data:')) or '/assets/' not in l or 'shared-assets' in l: continue
        p = os.path.normpath(os.path.join(base, urllib.parse.unquote(l))).replace(os.sep, '/')
        owner = p.split('/assets/')[0]
        if not (base + '/').startswith(owner + '/'):
            bad.append(f + ' -> ' + l + ' (shared-assets 이동 대상)')
print("FAIL:\n" + "\n".join(bad)) if bad else print("PASS: assets")
sys.exit(1 if bad else 0)
EOF
```

## 백로그·DO NOT

- `validated_on`은 사람이 포털에서 전체 실습을 완료하고 제공한 날짜만 기록한다. 자동 검사 통과를 E2E 완료로 표현하지 않는다.
- 소요 시간·행사 권한·비용·정리 완료 확인은 각 랩 TODO를 따른다. `status: draft`를 근거 없이 `active`로 바꾸지 않는다.
- Advanced의 NAT Gateway 단계 누락·VM 이름 불일치·SQL 서버 선택 값, 외부 앱과 패키지 호환성은 저자 확인 전까지 임의로 수정하지 않는다.
- 스크린샷의 계정·구독·개인 리소스 식별자는 자동 텍스트 검사로 검증되지 않는다. 공개 전 사람이 검토한다.
- 미참조 이미지도 삭제 승인 없이 제거하지 않는다. 검사를 생략·완화하거나 실패·미실행을 통과로 표시하지 않는다.
- [지원 범위](SUPPORT.md)와 문서 라이선스 전환은 보류 상태다. TODO가 남아 있는 상태를 공개 준비 완료로 표시하지 않는다.
