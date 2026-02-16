# Supabase Storage 설정 (로고·도장 이미지용)

견적서 링크를 열었을 때 로고와 도장 이미지가 정상 표시되려면 Supabase Storage 버킷을 만들어야 합니다.

> Storage가 없어도 견적서는 발행됩니다. 다만 이미지가 base64로 저장되어 다른 기기에서 깨질 수 있습니다.

## 1. Supabase 대시보드 접속

1. https://supabase.com/dashboard 에 로그인
2. 견적 시스템에서 사용 중인 프로젝트 선택

## 2. Storage 버킷 생성

1. 왼쪽 메뉴에서 **Storage** 클릭
2. **New bucket** 버튼 클릭
3. 설정:
   - **Name**: `quote-assets` (정확히 이 이름으로)
   - **Public bucket**: ✅ 체크
4. **Create bucket** 클릭

## 3. 업로드 권한 (RLS Policy)

1. `quote-assets` 버킷 옆 **⋮** 메뉴 → **Policies**
2. **New Policy** → **For full customization**
3. **Policy name**: `Allow public upload`
4. **Allowed operation**: `INSERT` 체크
5. **Policy definition**:
   - **USING**: `true`
   - **WITH CHECK**: `true`
6. **Target roles**: `anon` 체크 (브라우저에서 익명 업로드 허용)
7. **Create policy** 클릭

## 4. 자주 나오는 오류

| 오류 메시지 | 해결 |
|------------|------|
| Bucket not found / Object not found | 버킷 이름이 `quote-assets`인지 확인 |
| new row violates row-level security | 3번 RLS Policy 추가 |
| Permission denied | Public bucket 체크 + anon INSERT 정책 확인 |

## 5. 확인

버킷 생성 후 견적서를 발행해 보세요. 브라우저 개발자 도구(F12) → Console에서 "Storage 실패" 메시지가 없으면 성공입니다.
