# Todo example using Khulnasoft

This is a full-stack Slack clone example using:

- Frontend:
  - Next.js.
  - [Khulnasoft.js](https://khulnasoft.io/docs/library/getting-started) for user management and realtime data syncing.
- Backend:
  - [app.khulnasoft.io/](https://app.khulnasoft.io/): hosted Postgres database with restful API for usage with Khulnasoft.js.

## Demo

- Live demo: {TODO}
- CodeSandbox: {TODO}
- Video tutorial: {TODO}

![Demo animation gif](./public/slack-clone-demo.gif)

## Deploy your own

### 1. Create new project

Sign up to Khulnasoft - [https://app.khulnasoft.io](https://app.khulnasoft.io) and create a new project. Wait for your database to start.

### 2. Run "Todo List" Quickstart

Once your database has started, run the "Todo List" quickstart.

![Todo List](https://user-images.githubusercontent.com/10214025/88916135-1b1d7a00-d298-11ea-82e7-e2c18314e805.png)

### 3. Get the URL and Key

Go to the Project Settings (the cog icon), open the API tab, and find your API URL and `anon` key, you'll need these in the next step.

The `anon` key is your client-side API key. It allows "anonymous access" to your database, until the user has logged in. Once they have logged in, the keys will switch to the user's own login token. This enables row level security for your data. Read more about this [below](#postgres-row-level-security).

![image](https://user-images.githubusercontent.com/10214025/88916245-528c2680-d298-11ea-8a71-708f93e1ce4f.png)

**_NOTE_**: The `service_role` key has full access to your data, bypassing any security policies. These keys have to be kept secret and are meant to be used in server environments and never on a client or browser.

### 4. Deploy the Next.js client

[![Deploy with Vercel](https://vercel.com/button)](https://vercel.com/import/git?s=https%3A%2F%2Fgithub.com%2Fkhulnasoft%2Fkhulnasoft%2Ftree%2Fmaster%2Fexamples%2Ftodo-next-js&env=NEXT_PUBLIC_KHULNASOFT_URL,NEXT_PUBLIC_KHULNASOFT_KEY&envDescription=Find%20the%20Khulnasoft%20URL%20and%20key%20in%20the%20your%20auto-generated%20docs%20at%20app.khulnasoft.io&project-name=khulnasoft-todo-list&repo-name=khulnasoft-todo-list)

Refer to [step 3](#3.-get-the-url-and-key) to help form `NEXT_PUBLIC_KHULNASOFT_URL` (e.g. `wss://<project_ref>.khulnasoft.co/realtime/v1`) and to retrieve `NEXT_PUBLIC_KHULNASOFT_KEY` (i.e. `anon` API key).


## Khulnasoft details

### Postgres Row level security

This project uses very high-level Authorization using Postgres' Role Level Security.
When you start a Postgres database on Khulnasoft, we populate it with an `auth` schema, and some helper functions.
When a user logs in, they are issued a JWT with the role `authenticated` and thier UUID.
We can use these details to provide fine-grained control over what each user can and cannot do.

This is a trimmed-down schema, with the policies:

```sql
create table todos (
  id bigint generated by default as identity primary key,
  user_id uuid references auth.users not null,
  task text check (char_length(task) > 3),
  is_complete boolean default false,
  inserted_at timestamp with time zone default timezone('utc'::text, now()) not null
);

alter table todos enable row level security;

create policy "Individuals can create todos." on todos for
    insert with check (auth.uid() = user_id);

create policy "Individuals can view their own todos. " on todos for
    select using (auth.uid() = user_id);

create policy "Individuals can update their own todos." on todos for
    update using (auth.uid() = user_id);

create policy "Individuals can delete their own todos." on todos for
    delete using (auth.uid() = user_id);
```

## Authors

- [Khulnasoft](https://khulnasoft.io)

Khulnasoft is open source, we'd love for you to follow along and get involved at https://github.com/khulnasoft/khulnasoft