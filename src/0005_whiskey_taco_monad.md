# Whiskey Taco Monad

This post is in TypeScript.

## Dependency Injection

You are probably familiar with dependency injection. We see it in many forms.

```typescript
const fetchGitHubUser = (axios: AxiosInstance, metrics: Metrics) => (githubUsername: string): Promise<GitHubUser> => {
  ...
};

const saveGitHubUserToRedis = (redis: RedisClient, metrics: Metrics) => (user: GitHubUser): Promise<void> => {
  ...
};

async function fetchAndSave(axios: AxiosInstance, redis: RedisClient, metrics: Metrics) => (githubUsername: string): Promise<void> {
  const githubUser = await fetchGitHubUser(axios, metrics)(githubUsername);
  
  await saveGitHubUserToRedis(redis, metrics)(githubUser);  
}

function main() {
  const axios: AxiosInstance = ...;
  const redis: RedisClient = ...;
  const metrics: Metrics = ...;
  
  await fetchAndSave(axios, redis, metrics)(githubUsername);  
}
    
```

This is a little annoying...

This is better!

```typescript
const fetchGitHubUser = (githubUsername: string): Reader<{axios: AxiosInstance, metrics: Metrics}, Promise<GitHubUser>> => {
  ...
};

const saveGitHubUserToRedis = (user: GitHubUser): Reader<{redis: RedisClient, metrics: Metrics}, Promise<void>> => {
  ...
};

function fetchAndSave(githubUsername: string): Reader<{axios: AxiosInstance, redis: RedisClient, metrics: Metrics}, Promise<void>> {
  return fetchGitHubUser(githubUsername)
      .chain(githubUser => saveGitHubUserToRedis(githubUser));
}

function main() {
  const axios: AxiosInstance = ...;
  const redis: RedisClient = ...;
  const metrics: Metrics = ...;
  
  await fetchAndSave(githubUsername)
      .run({axios, redis, metrics});
}
```



```typescript
type GitHubDependencies = {axios: AxiosInstance, redis: RedisClient, metrics: Metrics};

const github = {
  users: {
    search: (query: string): Reader<GitHubDependencies, GitHubUser>
  }.
};
```
