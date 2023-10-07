<p align="center">
  <a href="http://nestjs.com/" target="blank"><img src="https://nestjs.com/img/logo-small.svg" width="200" alt="Nest Logo" /></a>
</p>

[circleci-image]: https://img.shields.io/circleci/build/github/nestjs/nest/master?token=abc123def456
[circleci-url]: https://circleci.com/gh/nestjs/nest

  <p align="center">A progressive <a href="http://nodejs.org" target="_blank">Node.js</a> framework for building efficient and scalable server-side applications.</p>
    <p align="center">
<a href="https://www.npmjs.com/~nestjscore" target="_blank"><img src="https://img.shields.io/npm/v/@nestjs/core.svg" alt="NPM Version" /></a>
<a href="https://www.npmjs.com/~nestjscore" target="_blank"><img src="https://img.shields.io/npm/l/@nestjs/core.svg" alt="Package License" /></a>
<a href="https://www.npmjs.com/~nestjscore" target="_blank"><img src="https://img.shields.io/npm/dm/@nestjs/common.svg" alt="NPM Downloads" /></a>
<a href="https://circleci.com/gh/nestjs/nest" target="_blank"><img src="https://img.shields.io/circleci/build/github/nestjs/nest/master" alt="CircleCI" /></a>
<a href="https://coveralls.io/github/nestjs/nest?branch=master" target="_blank"><img src="https://coveralls.io/repos/github/nestjs/nest/badge.svg?branch=master#9" alt="Coverage" /></a>
<a href="https://discord.gg/G7Qnnhy" target="_blank"><img src="https://img.shields.io/badge/discord-online-brightgreen.svg" alt="Discord"/></a>
<a href="https://opencollective.com/nest#backer" target="_blank"><img src="https://opencollective.com/nest/backers/badge.svg" alt="Backers on Open Collective" /></a>
<a href="https://opencollective.com/nest#sponsor" target="_blank"><img src="https://opencollective.com/nest/sponsors/badge.svg" alt="Sponsors on Open Collective" /></a>
  <a href="https://paypal.me/kamilmysliwiec" target="_blank"><img src="https://img.shields.io/badge/Donate-PayPal-ff3f59.svg"/></a>
    <a href="https://opencollective.com/nest#sponsor"  target="_blank"><img src="https://img.shields.io/badge/Support%20us-Open%20Collective-41B883.svg" alt="Support us"></a>
  <a href="https://twitter.com/nestframework" target="_blank"><img src="https://img.shields.io/twitter/follow/nestframework.svg?style=social&label=Follow"></a>
</p>
  <!--[![Backers on Open Collective](https://opencollective.com/nest/backers/badge.svg)](https://opencollective.com/nest#backer)
  [![Sponsors on Open Collective](https://opencollective.com/nest/sponsors/badge.svg)](https://opencollective.com/nest#sponsor)-->

## Description

[Nest](https://github.com/nestjs/nest) framework TypeScript starter repository.

## Installation

```bash
$ npm install
```

## Running the app

```bash
# development
$ npm run start

# watch mode
$ npm run start:dev

# production mode
$ npm run start:prod
```

## Test

```bash
# unit tests
$ npm run test

# e2e tests
$ npm run test:e2e

# test coverage
$ npm run test:cov
```

## Support

Nest is an MIT-licensed open source project. It can grow thanks to the sponsors and support by the amazing backers. If you'd like to join them, please [read more here](https://docs.nestjs.com/support).

## Stay in touch

- Author - [Kamil Myśliwiec](https://kamilmysliwiec.com)
- Website - [https://nestjs.com](https://nestjs.com/)
- Twitter - [@nestframework](https://twitter.com/nestframework)

## License

Nest is [MIT licensed](LICENSE).

// ▪️ Terminal - Start the REPL
npm run start -- --entryFile repl

// Get UserRepository and update User ID 1 (make sure you have a User with that ID of course)
await get("UserRepository").update({ id: 1 }, { role: 'regular' })
// Get all Users from the DB
await get("UserRepository").find()

// ▪️ Terminal - switch to REPL window and update User 1's Role (or whatever user you were working on)
await get("UserRepository").update({ id: 1 }, { role: 'admin' })

// -------
// ▪️Terminal - run REPL
npm run start -- --entryFile repl

// ⚠️ NOTE: If you’re getting any errors like “Property "permissions" was not found in "User"”
// you need to restart your REPL session (close the window & run it again).

// ⚙️ Let's get the UserRepository again, update user 1, and add the permissions create_coffee to it.
await get("UserRepository").update({ id: 1 }, { permissions: ['create_course'] })

API_KEYS

// ▪️ Terminal - Generate ApiKey Entity
nest g class users/api-keys/entities/api-key.entity --no-spec

// -----
// 📝 api-key.entity.ts - NEW FILE
import { Column, Entity, ManyToOne, PrimaryGeneratedColumn } from 'typeorm';
import { User } from '../../entities/user.entity';

@Entity()
export class ApiKey {
@PrimaryGeneratedColumn()
id: number;

@Column()
key: string;

@Column()
uuid: string;

@ManyToOne((type) => User, (user) => user.apiKeys)
user: User; // 👈 relationship with User Entity

// NOTE: As an exercise, you could create a new entity called "Scope" and establish
// a Many-to-Many relationship between API Keys and Scopes. With the scopes feature, users
// could selectively grant specific permissions/scopes to given API Keys. For example, some
// API Keys could only let 3-rd party applications "Read" data, but not modify it, etc.
// @Column()
// scopes: Scope[];
}

// -----
// 📝 UPDATES/ADDITIONS - users.module.ts - Add ApiKey for TypeORM
imports: [TypeOrmModule.forFeature([
User,
ApiKey, // 👈
])
],

// -----
// 📝 UPDATES/ADDITIONS - user.entity.ts - create relationship with ApiKey Entity
export class UserEntity {
// ...
@JoinTable() // 👈
@OneToMany((type) => ApiKey, (apiKey) => apiKey.users)
apiKeys: ApiKey[];
}

// ----
// ▪️ Terminal - create ApiKeysService
nest g service iam/authentication/api-keys --flat

// -----
// 📝 api-keys.service.ts - NEW FILE - FINAL RESULT
import { Injectable } from '@nestjs/common';
import { randomUUID } from 'crypto';
import { HashingService } from '../hashing/hashing.service';

export interface GeneratedApiKeyPayload { // ⚠️ note ideally this should be its own file, putting it here just for brevity
apiKey: string;
hashedKey: string;
}

@Injectable()
export class ApiKeysService {
constructor(private readonly hashingService: HashingService) {}

async createAndHash(id: number): Promise<GeneratedApiKeyPayload> {
const apiKey = this.generateApiKey(id);
const hashedKey = await this.hashingService.hash(apiKey);
return { apiKey, hashedKey };
}

async validate(apiKey: string, hashedKey: string): Promise<boolean> {
return this.hashingService.compare(apiKey, hashedKey);
}

extractIdFromApiKey(apiKey: string): string {
const [id] = Buffer.from(apiKey, 'base64').toString('ascii').split(' ');
return id;
}

private generateApiKey(id: number): string {
const apiKey = `${id} ${randomUUID()}`;
return Buffer.from(apiKey).toString('base64');
}
}

// ----
// ▪️ Terminal - Create ApiKeyGuard
nest g guard iam/authentication/guards/api-key

// -----
// 📝 api-key.guard.ts - NEW FILE - FINAL RESULT
import {
CanActivate,
ExecutionContext,
Injectable,
UnauthorizedException,
} from '@nestjs/common';
import { Observable } from 'rxjs';
import { Request } from 'express';
import { ApiKeysService } from '../api-keys.service';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { ApiKey } from '../../../users/api-keys/entities/api-key.entity';
import { REQUEST_USER_KEY } from '../../iam.constants';
import { ActiveUserData } from '../../interfaces/active-user-data.interface';

@Injectable()
export class ApiKeyGuard implements CanActivate {
constructor(
private readonly apiKeysService: ApiKeysService,
@InjectRepository(ApiKey)
private readonly apiKeysRepository: Repository<ApiKey>,
) {}

async canActivate(context: ExecutionContext): Promise<boolean> {
const request = context.switchToHttp().getRequest();
const apiKey = this.extractKeyFromHeader(request);
if (!apiKey) {
throw new UnauthorizedException();
}
const apiKeyEntityId = this.apiKeysService.extractIdFromApiKey(apiKey);
try {
const apiKeyEntity = await this.apiKeysRepository.findOne({
where: { uuid: apiKeyEntityId },
relations: { user: true },
});
await this.apiKeysService.validate(apiKey, apiKeyEntity.key);
request[REQUEST_USER_KEY] = {
sub: apiKeyEntity.user.id,
email: apiKeyEntity.user.email,
role: apiKeyEntity.user.role,
permissions: apiKeyEntity.user.permissions,
} as ActiveUserData;
} catch {
throw new UnauthorizedException();
}
return true;
}

private extractKeyFromHeader(request: Request): string | undefined {
const [type, key] = request.headers.authorization?.split(' ') ?? [];
return type === 'ApiKey' ? key : undefined;
}
}

// -----
// 📝 UPDATES/ADDITIONS - auth-type.enum.ts - Add new ApiKey type
export enum AuthType {
Bearer,
ApiKey, // 👈
None,
}

// -----
// 📝 UPDATES/ADDITIONS - authentication.guard.ts - make sure to add ApiKeyGuard here as well
export class AuthenticationGuard implements CanActivcate {
// ...
private readonly authTypeGuardMap: Record<
AuthType,
CanActivate | CanActivate[]

> = {

    [AuthType.Bearer]: this.accessTokenGuard,
    [AuthType.ApiKey]: this.apiKeyGuard, // 👈 add new apiKeyGuard
    [AuthType.None]: { canActivate: () => true },

};
}

// ----- 📯
// 📝 UPDATES/ADDITIONS - coffees.controller.ts - let's test everything out! Add the AuthType.ApiKey to the controller class
@Auth(AuthType.Bearer, AuthType.ApiKey) // 👈

// ----- 🟢 Testing everything out 🟢
// ▪️ Terminal - Start the NestJS REPL again
npm run start -- --entryFile repl
// declare unique id for our ApiKey
uuid = 'random_unique_id'
// await and retrieve ApiKeysService call createAndHash method to generate the ApiKeyPayload, save everything to a variable
payload = await get(ApiKeysService).createAndHash(uuid)
// And lastly, let’s insert a new ApiKey into our database table using the ApiKeysRepository by calling the save method on it
// - making sure to pass down uuid, a "key" with the value of our payload.hashedKey, and user (making sure to specify the
// user, _in our case_: id 1 (one)
await get("ApiKeyRepository").save({ uuid, key: payload.hashedKey, user: { id: 1 }})
// COPY the generated API KEY you get back
// ----
// head over to Insomnia and while on the /coffees endpoint page, let's switch to the Headers table and replace the current
// Authorization header value to ApiKey (making sure to add a space and then let's paste that generate Api Key here)
// ie: (something like) - ⚠️ NOTE your Api Key will be different then this one of course!
“ApiKey cmFuZG9tX3VuaXF1ZV9pZCAxOTg3NDk1MC0wZGU4LTQyNGUtOTNmOC0zYmQ4ZmY4ZDBhYzM=”
