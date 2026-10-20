# minhas-despesas-01
create table if not exists public.user_months (
  user_id uuid not null references auth.users(id) on delete cascade,
  month_key text not null check (month_key ~ '^[0-9]{4}-(0[1-9]|1[0-2])$'),
  data jsonb not null default '{}'::jsonb,
  updated_at timestamptz not null default now(),
  primary key (user_id, month_key)
);

alter table public.user_months enable row level security;

revoke all on table public.user_months
from anon, authenticated;

grant select, insert, update
on public.user_months to authenticated;

drop policy if exists "finance_select_owner"
on public.user_months;
drop policy if exists "finance_insert_owner"
on public.user_months;
drop policy if exists "finance_update_owner"
on public.user_months;

create policy "finance_select_owner"
on public.user_months
for select to authenticated
using ((select auth.uid()) = user_id);

create policy "finance_insert_owner"
on public.user_months
for insert to authenticated
with check ((select auth.uid()) = user_id);

create policy "finance_update_owner"
on public.user_months
for update to authenticated
using ((select auth.uid()) = user_id)
with check ((select auth.uid()) = user_id);
