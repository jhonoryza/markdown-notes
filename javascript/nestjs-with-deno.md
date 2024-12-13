## Nestjs with Deno 2

first lets run

```bash
deno init
```

update `deno.json` file add this line

```json
"compilerOptions": {
  "emitDecoratorMetadata": true,
  "experimentalDecorators": true
},
"nodeModulesDir": "auto",
"tasks": {
  "dev": "deno run --watch --allow-read --allow-env --allow-net main.ts"
},
"imports": {
  "@nestjs/common": "npm:@nestjs/common@^10.4.7",
  "@nestjs/core": "npm:@nestjs/core@^10.4.7",
  "@nestjs/platform-express": "npm:@nestjs/platform-express@^10.4.7"
},
"deploy": {

  "entrypoint": "main.ts"
}
```

create `services/HelloService.ts` file

```js
import {Injectable} from '@nestjs/common'

@Injectable()
export class HelloService {
    getHello() {
        return "hello world";
    }
}
```

create `controllers/HelloController.ts` file

```js
import {HelloService} from "../services/HelloService.ts";
import { Get, Controller } from "@nestjs/common"

@Controller()
export class HelloController {
    constructor(private helloService: HelloService) {}

    @Get('/')
    hello() {
        return this.helloService.getHello();
    }
}
```

create `main.ts` file

```js
import { NestFactory } from "@nestjs/core";
import { Module } from "@nestjs/common";
import "@nestjs/platform-express";
import { HelloService } from "./services/HelloService.ts";
import { HelloController } from "./controllers/HelloController.ts";

@Module({
    providers: [HelloService],
    controllers: [HelloController],
})
class AppModule {}

const app = await NestFactory.create(AppModule);
app.listen(3000);
```

lastly run and open [http://localhost:3000](http://localhost:3000)

```bash
npm run dev
```

thats it now you just run nestjs with deno 2
