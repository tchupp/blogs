# Whiskey Taco Monad

This post is in TypeScript.

## Dependency Injection

You are probably familiar with dependency injection. We see it in many forms.

```
const fetchGithubUser = (axios: AxiosInstance, metrics: Metrics) => (githubUsername: string): Promise<GithubUser> => {
  ...
};

const saveGithubUserToRedis = (redis: RedisClient, metrics: Metrics) => (user: GithubUser): Promise<void> => {
  ...
};

async function fetchAndSave(axios: AxiosInstance, redis: RedisClient, metrics: Metrics): Promise<void> {
  const githubUser = await fetchGithubUser(axios, metrics)("...");
  
  await saveGithubUserToRedis(redis, metrics)(githubUser);  
}

function main() {
  const axios: AxiosInstance = ...;
  const redis: RedisClient = ...;
  const metrics: Metrics = ...;
  
  await fetchAndSave(axios, redis, metrics)(githubUser);  
}
    
```

This is a little annoying
