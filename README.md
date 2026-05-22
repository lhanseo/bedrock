📊 프로젝트 통합 코드 리뷰 리포트

분석 범위: C:\final_project 현재 폴더 기준. worktrees, _upstream-online-boutique, generated proto, lockfile은 제외하고 실제 유지보수 대상 코드/IaC 중심으로 분석했습니다.

📄 online-boutique-dr-app/src/cartservice/src/cartstore/AlloyDBCartStore.cs (177줄)
🔹 스타일 검사 결과
발견된 스타일 위반 사항은 다음과 같습니다:

[62라인] 위반 유형: 들여쓰기 불일치
AddItemAsync 메서드가 클래스 내부 들여쓰기와 맞지 않습니다.

[64, 106, 143라인] 위반 유형: 직접 Console 로깅
운영 코드에서 Console.WriteLine으로 userId를 출력합니다. 구조화 로거 사용이 권장됩니다.

[9093, 127130, 151~154라인] 위반 유형: 불필요한 Task.Run
이미 async API를 사용하고 있어 Task.Run(() => ExecuteNonQueryAsync()) 패턴은 불필요합니다.

🔹 보안 취약점 검사 결과
⚠️ SQL Injection 발견
위치: 69, 81~86, 113, 148라인
userId, productId, tableName이 SQL 문자열에 직접 삽입됩니다. NpgsqlParameter와 테이블명 allowlist 검증이 필요합니다.

⚠️ 하드코딩된 권한 계정
위치: 44라인
postgres 슈퍼유저 계정을 애플리케이션 계정으로 사용합니다. 최소 권한 DB 계정으로 분리해야 합니다.

⚠️ 정보 노출
위치: 9899, 134135, 159~160라인
예외 전체를 gRPC 응답에 포함합니다. 내부 DB 오류/연결 정보가 노출될 수 있습니다.

📄 online-boutique-dr-app/src/productcatalogservice/catalog_loader.go (161줄)
🔹 스타일 검사 결과
[106라인] 위반 유형: 불필요한 cleanup 래퍼
cleanup := func() error { return dialer.Close() } 대신 defer dialer.Close()가 간결합니다.

[132라인] 위반 유형: 문자열 결합 SQL
쿼리 구성 방식이 타입 안전하지 않고 검증 흐름이 약합니다.

[141~157라인] 위반 유형: rows.Err() 누락
반복 종료 후 rows.Err() 확인이 없어 scan 외 오류를 놓칠 수 있습니다.

🔹 보안 취약점 검사 결과
⚠️ SQL Injection 가능성
위치: 93, 132라인
ALLOYDB_TABLE_NAME 환경 변수를 검증 없이 SQL에 결합합니다. 운영자 입력이라도 allowlist가 필요합니다.

⚠️ DB 연결 보안 약화
위치: 110라인
sslmode=disable이 설정되어 있습니다. 네트워크 경계 밖으로 확장될 가능성이 있으면 TLS 사용이 필요합니다.

✅ 하드코딩된 비밀번호
비밀번호는 Secret Manager에서 읽으므로 직접 하드코딩은 발견되지 않았습니다.

📄 online-boutique-dr-app/src/shoppingassistantservice/shoppingassistantservice.py (120줄)
🔹 스타일 검사 결과
[20~23라인] 위반 유형: import 순서 불일치
표준 라이브러리, 서드파티 import가 PEP8 순서와 다릅니다.

[66라인] 위반 유형: 함수 네이밍 컨벤션
talkToGemini는 Python 컨벤션상 talk_to_gemini가 권장됩니다.

[83~107라인] 위반 유형: 과도한 print 디버깅
프롬프트, 응답, 검색 결과가 표준 출력에 남습니다.

🔹 보안 취약점 검사 결과
⚠️ 입력 검증 미흡
위치: 68, 79라인
request.json['message'], request.json['image']를 검증 없이 LLM 요청에 사용합니다. 크기 제한, URL allowlist, 스키마 검증이 필요합니다.

⚠️ 프롬프트/개인정보 로그 노출
위치: 83~107라인
사용자 프롬프트와 모델 응답이 로그에 남습니다.

⚠️ 인증/레이트리밋 부재
위치: 65, 120라인
POST 엔드포인트가 인증 없이 열릴 경우 LLM 비용 남용 위험이 있습니다.

📄 online-boutique-dr-app/src/frontend/handlers.go (635줄)
🔹 스타일 검사 결과
[96, 194, 443, 484~485라인] 위반 유형: fmt 직접 출력
운영 핸들러에서 fmt.Println/Printf 대신 요청 컨텍스트 로거 사용이 권장됩니다.

[101라인] 위반 유형: 항상 참인 조건
len(addrs) >= 0은 항상 참입니다. len(addrs) > 0가 의도에 맞습니다.

[447~448라인] 위반 유형: 응답 순서 오류
w.Write(jsonData) 후 w.WriteHeader(http.StatusOK)를 호출합니다. 헤더가 먼저 설정되어야 합니다.

🔹 보안 취약점 검사 결과
⚠️ Open Redirect 가능성
위치: 517~522라인
Referer 헤더를 그대로 Location에 사용합니다. 동일 origin 검증 또는 고정 경로 리다이렉션이 필요합니다.

⚠️ 정보 노출
위치: 538~545라인
내부 오류 문자열을 사용자 화면에 렌더링합니다. 사용자 메시지와 서버 로그를 분리해야 합니다.

⚠️ 요청 본문 크기 제한 누락
위치: 465~478라인
챗봇 프록시가 요청/응답 본문 크기를 제한하지 않습니다. http.MaxBytesReader 및 HTTP client timeout이 필요합니다.

📄 online-boutique-dr-app/src/frontend/middleware.go (111줄)
🔹 스타일 검사 결과
[21라인] 위반 유형: import 정렬 불일치
os import 위치가 gofmt 기준 정렬과 맞지 않습니다.

[90~92라인] 위반 유형: 매직 값 사용
공유 세션 ID가 코드에 직접 박혀 있습니다.

🔹 보안 취약점 검사 결과
⚠️ 쿠키 보안 속성 누락
위치: 97~101라인
세션 쿠키에 HttpOnly, Secure, SameSite가 없습니다.

⚠️ 고정 세션 ID 옵션
위치: 90~92라인
ENABLE_SINGLE_SHARED_SESSION=true일 때 모든 사용자가 같은 세션을 공유합니다. 데모 전용 옵션으로 제한해야 합니다.

📄 online-boutique-dr-app/src/frontend/main.go (235줄)
🔹 스타일 검사 결과
[132~139라인] 위반 유형: 환경 변수 필수값 하드 실패
누락 시 panic으로 종료됩니다. 설정 검증 단계에서 명확한 오류를 반환하는 방식이 좋습니다.

🔹 보안 취약점 검사 결과
⚠️ 평문 HTTP/gRPC 사용
위치: 171, 229~231라인
http.ListenAndServe와 insecure.NewCredentials()를 사용합니다. 내부망 전제라도 mTLS/Ingress TLS 경계가 필요합니다.

⚠️ 내부 서비스 주소 노출 가능성
위치: 232~233라인
연결 실패 시 내부 주소가 panic 메시지에 포함됩니다.

📄 online-boutique-dr-app/src/frontend/packaging_info.go (79줄)
🔹 스타일 검사 결과
[20, 66라인] 위반 유형: deprecated 패키지 사용
io/ioutil 대신 io.ReadAll 사용이 권장됩니다.

[53라인] 위반 유형: 직접 출력
요청 URL을 fmt.Println으로 출력합니다.

🔹 보안 취약점 검사 결과
⚠️ SSRF/내부 요청 위험
위치: 43, 52~54라인
PACKAGING_SERVICE_URL과 productId를 조합해 서버에서 HTTP 요청을 보냅니다. URL allowlist와 timeout이 필요합니다.

📄 online-boutique-dr-app/src/paymentservice/server.js (106줄)
🔹 스타일 검사 결과
[21, 58~59라인] 위반 유형: 세미콜론 누락
프로젝트가 semicolon 스타일이면 일관성이 깨집니다.

[63라인] 위반 유형: 익명 콜백
디버깅 편의상 이름 있는 콜백 또는 화살표 함수 통일이 권장됩니다.

🔹 보안 취약점 검사 결과
⚠️ 결제 정보 로그 노출
위치: 43라인
JSON.stringify(call.request)로 카드번호/CVV가 로그에 남을 수 있습니다. 결제 필드는 마스킹해야 합니다.

⚠️ 평문 gRPC
위치: 62라인
grpc.ServerCredentials.createInsecure()를 사용합니다.

📄 online-boutique-dr-app/src/paymentservice/charge.js (86줄)
🔹 스타일 검사 결과
[38, 44, 50라인] 위반 유형: 미사용 파라미터
생성자 파라미터가 일부 사용되지 않습니다.

🔹 보안 취약점 검사 결과
✅ SQL Injection
DB 접근 코드가 없어 해당 없음.

⚠️ 민감정보 부분 노출
위치: 51, 82라인
카드 마지막 4자리와 결제 금액을 로그/오류에 포함합니다. 운영 로그 정책에 따라 마스킹 기준을 명확히 해야 합니다.

📄 online-boutique-dr-app/src/currencyservice/server.js (198줄)
🔹 스타일 검사 결과
[28, 50라인] 위반 유형: 공백 스타일
if(process.env...) 형태로 공백이 누락되었습니다.

[62라인] 위반 유형: 객체 리터럴 공백
{url: collectorUrl} 대신 { url: collectorUrl }가 일반적입니다.

🔹 보안 취약점 검사 결과
⚠️ 입력 검증 미흡
위치: 146~155라인
지원하지 않는 통화 코드가 들어오면 undefined 연산으로 잘못된 결과가 발생할 수 있습니다.

⚠️ 평문 gRPC
위치: 190라인
내부 서비스라도 mTLS 또는 네트워크 정책으로 보완해야 합니다.

📄 online-boutique-dr-app/src/checkoutservice/main.go (394줄)
🔹 스타일 검사 결과
[151라인] 위반 유형: TODO 포맷
//TODO 대신 // TODO: 형식이 Go 컨벤션에 더 맞습니다.

[360라인] 위반 유형: 전달받은 context 미사용
context.TODO() 대신 인자로 받은 ctx를 사용해야 취소/타임아웃이 전파됩니다.

[263라인] 위반 유형: 오류 무시
_ = cs.emptyUserCart(...)로 장바구니 비우기 실패를 무시합니다.

🔹 보안 취약점 검사 결과
⚠️ 평문 gRPC 클라이언트
위치: 214~216라인
insecure.NewCredentials() 사용.

⚠️ 내부 오류 노출
위치: 254, 260라인
결제/배송 오류 상세가 응답에 포함될 수 있습니다.

📄 online-boutique-dr-app/src/shippingservice/main.go (183줄)
🔹 스타일 검사 결과
[155~160라인] 위반 유형: 미구현 TODO 잔존
Tracing/Stats TODO가 운영 코드에 남아 있습니다.

🔹 보안 취약점 검사 결과
⚠️ gRPC Reflection 노출
위치: 98라인
운영 환경에서 reflection은 서비스 메서드 노출면을 넓힙니다.

⚠️ nil 입력 검증 미흡
위치: 146라인
in.Address nil 여부를 확인하지 않아 잘못된 요청으로 panic 가능성이 있습니다.

📄 online-boutique-dr-app/src/productcatalogservice/product_catalog.go (83줄)
🔹 스타일 검사 결과
[75~79라인] 위반 유형: 오류 무시성 흐름
카탈로그 reload 실패 시 빈 배열만 반환하여 장애 원인 파악이 어렵습니다.

🔹 보안 취약점 검사 결과
⚠️ 입력 길이 제한 없음
위치: 60~67라인
검색어 길이 제한이 없어 큰 입력으로 CPU 사용량을 늘릴 수 있습니다.

⚠️ 정보 노출
위치: 57라인
존재하지 않는 product ID를 오류 메시지에 그대로 포함합니다.

📄 multi-cloud-dr-infra/terraform/modules/aws/network/main.tf (210줄)
🔹 스타일 검사 결과
구조와 Terraform 포맷은 전반적으로 양호합니다.

🔹 보안 취약점 검사 결과
⚠️ SSH 전체 공개
위치: 142~148라인
Bastion SSH가 0.0.0.0/0에 열려 있습니다. 관리자 CIDR 변수로 제한해야 합니다.

⚠️ 광범위 egress
위치: 150155, 167172, 200~205라인
모든 outbound 허용은 운영 기준에서 최소화 검토가 필요합니다.

📄 multi-cloud-dr-infra/terraform/modules/azure/network/main.tf (67줄)
🔹 스타일 검사 결과
[1~13라인] 리소스 블록 내부 공백이 다소 과합니다. terraform fmt 기준 정렬이 권장됩니다.

🔹 보안 취약점 검사 결과
⚠️ VM 패스워드 로그인 허용
위치: 23~26라인
disable_password_authentication = false입니다. AAD SSH 확장을 쓰고 있으므로 비밀번호 인증은 끄는 것이 안전합니다.

✅ 하드코딩된 비밀번호
비밀번호 값 자체는 변수로 주입되며 파일에 직접 하드코딩되어 있지는 않습니다.

📄 multi-cloud-dr-infra/terraform/envs/dev/aws/main.tf (170줄)
🔹 스타일 검사 결과
[87~116라인] 위반 유형: 하드코딩된 IAM Principal
계정 ID와 사용자/역할 ARN이 환경 코드에 직접 박혀 있습니다. 변수화가 권장됩니다.

🔹 보안 취약점 검사 결과
⚠️ 과도한 EKS 관리자 권한
위치: 94102, 112120라인
사용자와 GitHub Actions 역할에 AmazonEKSClusterAdminPolicy가 클러스터 전체 범위로 부여됩니다.

✅ 하드코딩된 비밀번호
DB 비밀번호는 sensitive variable 및 Secrets Manager로 전달되며 직접 평문 저장은 보이지 않습니다.

📋 종합 요약

취약점 유형 발견 여부 심각도
SQL Injection ⚠️ 있음 높음
XSS 직접 위험 ❌ 명확한 직접 위험 없음 -
하드코딩된 비밀번호 ❌ 없음 -
민감정보 로그 노출 ⚠️ 있음 높음
Open Redirect ⚠️ 있음 중간
쿠키 보안 속성 누락 ⚠️ 있음 중간
평문 gRPC/HTTP ⚠️ 있음 중간
IaC 보안그룹 과다 공개 ⚠️ 있음 높음

참고: 현재 환경에 go/gofmt가 설치되어 있지 않아 실제 빌드/포맷 명령 검증은 수행하지 못했습니다.
