# 견적서 링크 보안 강화 (share_token)

순차 ID(id=24, id=25...)로 다른 고객 견적서를 추측해 볼 수 있는 문제를 해결합니다.
예측 불가능한 UUID 토큰을 사용해 `?token=abc123-def456-...` 형식으로 변경합니다.

## Supabase 설정 (1회만 실행)

1. Supabase 대시보드 → **SQL Editor**
2. 아래 SQL 실행:

```sql
-- share_token 컬럼 추가 (기존 quotes 테이블)
ALTER TABLE quotes ADD COLUMN IF NOT EXISTS share_token TEXT UNIQUE;

-- 기존 데이터에 토큰 생성 (이미 있으면 스킵)
UPDATE quotes 
SET share_token = gen_random_uuid()::text 
WHERE share_token IS NULL;

-- 인덱스 추가 (조회 속도)
CREATE INDEX IF NOT EXISTS idx_quotes_share_token ON quotes(share_token);
```

3. **Run** 클릭

## 적용 후

- 새로 발행하는 견적서: `?token=550e8400-e29b-41d4-a716-446655440000` 형식
- 기존 링크(?id=24): 계속 동작 (하위 호환)
- 숫자 바꿔가며 접근 불가
