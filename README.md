[SQL_Supabase_Carrera_Zombi_V8.sql](https://github.com/user-attachments/files/33940561/SQL_Supabase_Carrera_Zombi_V8.sql)
-- SUPABASE: esquema para el modo multijugador de la Carrera Zombi V8
-- En Supabase abre SQL Editor, pega este archivo completo y pulsa Run.
-- NOTA: estas políticas abiertas son SOLO para una prueba académica/demo.
-- Para producción se recomienda mover validación y puntuación a funciones seguras
-- (Edge Functions/RPC) y limitar las políticas para impedir suplantación o trampas.

create table if not exists public.zombie_rooms (
  code text primary key,
  host_token text not null,
  status text not null default 'lobby' check (status in ('lobby','running','finished')),
  question_index integer not null default 0 check (question_index between 0 and 9),
  created_at timestamptz not null default now(),
  started_at timestamptz,
  finished_at timestamptz
);

create table if not exists public.zombie_players (
  id uuid primary key default gen_random_uuid(),
  room_code text not null references public.zombie_rooms(code) on delete cascade,
  player_name text not null,
  token text not null,
  avatar text not null default '🧟',
  position integer not null default 0 check (position between 0 and 10),
  score integer not null default 0,
  answered_index integer not null default -1,
  last_correct boolean,
  finished boolean not null default false,
  joined_at timestamptz not null default now()
);

create index if not exists zombie_players_room_rank_idx
  on public.zombie_players(room_code, position desc, score desc);

alter table public.zombie_rooms enable row level security;
alter table public.zombie_players enable row level security;

drop policy if exists "demo public room access" on public.zombie_rooms;
create policy "demo public room access" on public.zombie_rooms
  for all to anon, authenticated using (true) with check (true);

drop policy if exists "demo public player access" on public.zombie_players;
create policy "demo public player access" on public.zombie_players
  for all to anon, authenticated using (true) with check (true);

-- Añadir ambas tablas a Realtime. Ejecuta estas líneas una sola vez.
alter publication supabase_realtime add table public.zombie_rooms;
alter publication supabase_realtime add table public.zombie_players;
