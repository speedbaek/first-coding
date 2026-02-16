# 특허법인 테헤란 스마트 견적 시스템 v4.0 — 1차 완성본

**기준일**: 2025년 2월  
**배포 URL**: https://teheran-patent.netlify.app/

## 복원 안내
나중에 오류가 많아지면 **이 시점의 index.html**로 되돌아오세요.

## 1차 완성본 주요 기능
- ✅ 견적일/견적 유효기간 자동 표시 (오늘 날짜 + 7일)
- ✅ 로고/도장 이미지 업로드 (localStorage + Supabase Storage)
- ✅ 견적서 발행 및 링크 복사 (share_token UUID 암호화)
- ✅ 기본 예시 데이터 (김진왕 원장, 비용 항목 등)
- ✅ Supabase try-catch로 초기화 오류 시에도 UI 동작
- ✅ 파일 업로드 버튼 (absolute inset-0 opacity-0 패턴)
- ✅ checkUrlAndLoad 시 빈 데이터 기본값 유지

## 관련 파일
- `index.html` — 메인 (단일 HTML)
- `STORAGE_SETUP.md` — Supabase Storage 설정
- `SUPABASE_SHARE_TOKEN.md` — share_token 컬럼 추가
