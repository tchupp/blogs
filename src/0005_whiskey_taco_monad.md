# Whiskey Taco Monad

This post is in TypeScript.

## Dependency Injection

You are probably familiar with dependency injection. We see it in many forms.

```typescript
const fetchGithubUser = (axios: AxiosInstance, metrics: Metrics) => (githubUsername: string): Promise<GithubUser> => {
  ...
};

const saveGithubUserToRedis = (redis: RedisClient, metrics: Metrics) => (user: GithubUser): Promise<void> => {
  ...
};

async function fetchAndSave(axios: AxiosInstance, redis: RedisClient, metrics: Metrics) => (githubUsername: string): Promise<void> {
  const githubUser = await fetchGithubUser(axios, metrics)(githubUsername);
  
  await saveGithubUserToRedis(redis, metrics)(githubUser);  
}

function main() {
  const axios: AxiosInstance = ...;
  const redis: RedisClient = ...;
  const metrics: Metrics = ...;
  
  await fetchAndSave(axios, redis, metrics)(githubUser);  
}
    
```

This is a little annoying...

This is better!

```typescript
const fetchGithubUser = (githubUsername: string): Reader<{axios: AxiosInstance, metrics: Metrics}, Promise<GithubUser>> => {
  ...
};

const saveGithubUserToRedis = (user: GithubUser): Reader<{redis: RedisClient, metrics: Metrics}, Promise<void>> => {
  ...
};

function fetchAndSave(githubUsername: string): Reader<{axios: AxiosInstance, redis: RedisClient, metrics: Metrics}, Promise<void>> {
  return fetchGithubUser(githubUsername)
      .chain(githubUser => saveGithubUserToRedis(githubUser));
}

function main() {
  const axios: AxiosInstance = ...;
  const redis: RedisClient = ...;
  const metrics: Metrics = ...;
  
  await fetchAndSave(githubUser)
      .run({axios, redis, metrics});  
}
```
