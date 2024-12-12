## Create a Lobby mechanism for Ticketing

in this article we will store the lobby data using redis server.

lets define how a lobby data contains, we can create `Lobby` for a lobby model

```ts
const PREFIX = "ticket";

export class Lobby {
  id: string;

  sessionId: string;

  users: string[];

  createdAt: string;
}
```

lets create a function let say `findOrCreateLobby`, we can store the lobby data
in redis server

this function will make sure only one user can only join one lobby

```ts
async findOrCreateLobby(userId: string, maxUser: number): Promise<Lobby> {
    // find active lobby
    const keys = await this.redisService.redis.keys(`lobby:${PREFIX}:*`);

    const lobbies = await Promise.all(keys.map(async (key) => {
      const data = await this.redisService.redis.hgetall(key).catch((err) => {
        this.logger.error(
          'ERROR GET LOBBY',
          { meta: { stack: err?.stack } },
          'REDIS',
        );
        throw new InternalServerErrorException('ERROR GET LOBBY', '500');
      });
      const currentUsers = JSON.parse(data.users || '[]');
      const lobbyData: Lobby = {
        id: data.id,
        sessionId: data.sessionId,
        users: currentUsers,
        createdAt: data.createdAt,
      };

      // if user already in lobby, return lobby data
      if (currentUsers.includes(userId)) {
        return lobbyData;
      }

      if (currentUsers.length < maxUser) {
        // if lobby is not full, lets add current user
        currentUsers.push(userId);
        await this.redisService.redis.hset(key, 'users', JSON.stringify(currentUsers)).catch((err) => {
          this.logger.error(
            'ERROR UPDATE LOBBY',
            { meta: { stack: err?.stack } },
            'REDIS',
          );
          throw new InternalServerErrorException('ERROR UPDATE LOBBY', '500');
        });
        return lobbyData;
      }

      return null; // if there is no matching lobby
    }));

    const lobby = lobbies
      .filter((value: Lobby | null) => value != null)
      .filter((value: Lobby | null) => value?.users.includes(userId))[0] || null;

    if (lobby != null) {
      return lobby;
    }

    // if there is no available lobby, lets create a new lobby
    const newLobby: Lobby = {
      id: ulid(),
      sessionId: ulid(),
      users: [userId],
      createdAt: Date.now().toString(),
    };
    await this.redisService.redis.hset(
      `lobby:${PREFIX}:${newLobby.id}`,
      'id',
      newLobby.id,
      'sessionId',
      newLobby.sessionId,
      'users',
      JSON.stringify(newLobby.users),
      'createdAt',
      newLobby.createdAt,
    ).catch((err) => {
      this.logger.error(
        'ERROR CREATE NEW LOBBY',
        { meta: { stack: err?.stack } },
        'REDIS',
      );
      throw new InternalServerErrorException('ERROR CREATE NEW LOBBY', '500');
    });
    return newLobby;
  }
```

lets create a function called `bookedTicket`, in this function we will retreive
lobby data and validate it

then after transaction is done, we can delete the lobby

```ts
async bookedTicket(lobbyId) {
    
    const usersString = await this.redisService.redis.hget(`lobby:${PREFIX}:${lobbyId}`, 'users');
    
    // validate valid lobby
    if (usersString == null || usersString == '' || usersString == undefined) {
      throw new NotFoundException(`LOBBY  ${lobbyId} NOT FOUND`, '500');
    }

    const users: string[] = JSON.parse(usersString);
    const lobby: Lobby = await this.redisService.redis.hgetall(`lobby:${PREFIX}:${lobbyId}`);

    // do what ever you want here
    // maybe create ticket transaction
    

    // delete current lobby
    await this.redisService.redis.del(`lobby:${PREFIX}:${lobbyId}`).catch((err) => {
      this.logger.error(
        'ERROR REMOVE LOBBY',
        { meta: { stack: err?.stack } },
        'REDIS',
      );
      throw new InternalServerErrorException('ERROR REMOVE LOBBY', '500');
    });

    return null;
}
```
