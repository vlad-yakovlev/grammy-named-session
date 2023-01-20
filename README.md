# grammy-named-session

Named session for grammY. Works the same way as [built-in session](https://grammy.dev/plugins/session.html) but allows you to set custom name for data access.

![GitHub CI](https://img.shields.io/github/actions/workflow/status/vlad-yakovlev/grammy-named-session/ci.yml?branch=main&label=github-ci)
[![Codecov](https://img.shields.io/codecov/c/github/vlad-yakovlev/grammy-named-session/main)](https://codecov.io/gh/vlad-yakovlev/grammy-named-session)
[![NPM](https://img.shields.io/npm/v/grammy-named-session)](https://www.npmjs.org/package/grammy-named-session)

## How to install

```sh
npm install grammy-named-session
```

## How to use

```ts
import { Bot, Context } from 'grammy';
import { namedSession } from 'grammy-named-session';

interface Item {
  id: string
  value: string
}

interface MyState {
  items: Item[]
  users: Record<number, string>
}

interface MyContext extends Context {
  myState: MyState
}

const myStateMiddleware = (dirName: string) => {
  return namedSession<MyContext, 'myState'>({
    // type MaybePromise<T> = Promise<T> | T

    // Initial data, required
    // (ctx: MyContext) => MaybePromise<MyContext['myState']>
    getInitial: () => ({
      items: [],
      users: {},
    }),

    // Session key, required
    // (ctx: MyContext) => MaybePromise<string | undefined>
    getSessionKey: (ctx) =>
      ctx.chat?.id === undefined ? undefined : `my_${ctx.chat.id}`,

    // Storage adapter, required
    // (ctx: MyContext) => MaybePromise<StorageAdapter<MyContext['myState']>>
    getStorage: () => new FileAdapter({ dirName }),

    // Access name, must be the same as in the types, required
    // string
    name: 'myState',
  })
}

(async () => {
  const bot = new Bot<MyContext>('<bot-token>');

  bot.use(myStateMiddleware('db/v1'));

  bot.command('start', async (ctx) => {
    // Access data as usual but with custom name
    await ctx.mySession.users[ctx.from.id] = ctx.from.first_name;
  });
})()
```
