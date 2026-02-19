
-- 1. Drop existing tables to start fresh (Reverse order of dependencies)
DROP TABLE IF EXISTS public.events CASCADE;
DROP TABLE IF EXISTS public.departments CASCADE;
DROP TABLE IF EXISTS public.messages CASCADE;
DROP TABLE IF EXISTS public.files CASCADE;
DROP TABLE IF EXISTS public.comments CASCADE;
DROP TABLE IF EXISTS public.posts CASCADE;
DROP TABLE IF EXISTS public.profiles CASCADE;

-- 2. Create tables

-- Create a table for public profiles
create table public.profiles (
  id uuid references auth.users not null primary key,
  email text unique not null,
  full_name text,
  avatar_url text,
  department text,
  year text,
  bio text,
  skills text[],
  role text default 'student' check (role in ('student', 'staff', 'admin')),
  updated_at timestamp with time zone,
  created_at timestamp with time zone default timezone('utc'::text, now())
);

-- Enable Row Level Security (RLS)
alter table public.profiles enable row level security;

create policy "Public profiles are viewable by everyone." on public.profiles
  for select using (true);

create policy "Users can insert their own profile." on public.profiles
  for insert with check (auth.uid() = id);

create policy "Users can update their own profile." on public.profiles
  for update using (auth.uid() = id);

-- Create a table for posts (Social Feed)
create table public.posts (
  id uuid default uuid_generate_v4() primary key,
  user_id uuid references public.profiles(id) not null,
  content text,
  image_url text,
  file_url text,
  likes integer default 0,
  created_at timestamp with time zone default timezone('utc'::text, now())
);

alter table public.posts enable row level security;

create policy "Posts are viewable by everyone." on public.posts
  for select using (true);

create policy "Users can create posts." on public.posts
  for insert with check (auth.uid() = user_id);

create policy "Users can update their own posts." on public.posts
  for update using (auth.uid() = user_id);

create policy "Users can delete their own posts." on public.posts
  for delete using (auth.uid() = user_id);

-- Create a table for comments
create table public.comments (
  id uuid default uuid_generate_v4() primary key,
  post_id uuid references public.posts(id) on delete cascade not null,
  user_id uuid references public.profiles(id) not null,
  content text not null,
  created_at timestamp with time zone default timezone('utc'::text, now())
);

alter table public.comments enable row level security;

create policy "Comments are viewable by everyone." on public.comments
  for select using (true);

create policy "Users can create comments." on public.comments
  for insert with check (auth.uid() = user_id);

-- Create a table for files (File Sharing System)
create table public.files (
  id uuid default uuid_generate_v4() primary key,
  uploader_id uuid references public.profiles(id) not null,
  title text not null,
  description text,
  file_url text not null,
  file_type text, -- PDF, PPT, DOCX, ZIP
  department text,
  semester text,
  subject text,
  downloads integer default 0,
  rating float default 0.0,
  created_at timestamp with time zone default timezone('utc'::text, now())
);

alter table public.files enable row level security;

create policy "Files are viewable by everyone." on public.files
  for select using (true);

create policy "Users can upload files." on public.files
  for insert with check (auth.uid() = uploader_id);

-- Create a table for messages (Messaging System)
create table public.messages (
  id uuid default uuid_generate_v4() primary key,
  sender_id uuid references public.profiles(id) not null,
  receiver_id uuid references public.profiles(id), 
  group_id uuid, -- For group chats
  content text,
  file_url text,
  is_read boolean default false,
  created_at timestamp with time zone default timezone('utc'::text, now())
);

alter table public.messages enable row level security;

create policy "Users can see their own messages." on public.messages
  for select using (auth.uid() = sender_id or auth.uid() = receiver_id);

create policy "Users can send messages." on public.messages
  for insert with check (auth.uid() = sender_id);

-- Create a table for Departments (Communities)
create table public.departments (
  id uuid default uuid_generate_v4() primary key,
  name text unique not null,
  description text,
  created_at timestamp with time zone default timezone('utc'::text, now())
);

alter table public.departments enable row level security;

create policy "Departments are viewable by everyone." on public.departments
  for select using (true);

-- Create a table for Department Events
create table public.events (
  id uuid default uuid_generate_v4() primary key,
  department_id uuid references public.departments(id),
  title text not null,
  description text,
  event_date timestamp with time zone,
  created_at timestamp with time zone default timezone('utc'::text, now())
);

alter table public.events enable row level security;

create policy "Events are viewable by everyone." on public.events
  for select using (true);
