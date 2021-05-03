# Apa ini

Demo zaruba dengan segenap kemampuan yang ada, jika kembali dari langit, semoga hidup kan jadi lebih baik.

# Kondisi awal

Sudah ada satu `node-service` yang dibuat dengan express-js.
Di dalamnya ada satu environment `template.env` dengan isi sebagai berikut:

```
PORT=3000
MODE=default
```


# Target

* Kita ingin punya `serviceMaster` yang code-base nya adalah `node-service` di port `3000` dengan mode `master`.
* Kita ingin punya `serviceWorker` yang code-base nya juga `node-service` di port `3001` dengan mode `worker`.
* Kita ingin punya `serviceAjaib` yang code-base nya bakal dibuat pakai fastAPI
* Di `serviceAjaib` kita mau handle URL `GET /info`
* Di `serviceAjaib` kita juga mau handle CRUD `engineer` dengan field `name` dan `role`
* `serviceAjaib` harus run di port `3002`
* run semuanya
* Tiba-tiba kita sadar bahwa serviceAjaib tidak boleh run sebelum ada mysql run
* Kita maau deploy pakai helm (serviceAjaib saja)
* Tiba-tiba kita sadar ada environment variable yang kurang di `serviceAjaib`, kita mau ada envvar `SALAM=oraUmum`

# Aksi

```bash
# bikin zaruba project
mkdir runner
cd runner
zaruba please initProject

# bikin runner untuk serviceMaster (supaya tidak mengetik sepanjang ini, cukup "zaruba please")
zaruba please makeNodeJsServiceTask generator.service.location=../node-service generator.service.name=serviceMaster generator.nodeJsService.startCommand="npm start" generator.nodeJsService.runnerVersion=node

# edit serviceMaster, ubah mode='master'
vim zaruba-tasks/serviceMaster.zaruba.yaml

# bikin runner untuk serviceWorker (supaya tidak mengetik sepanjang ini, cukup "zaruba please")
zaruba please makeNodeJsServiceTask generator.service.location=../node-service generator.service.name=serviceWorker generator.nodeJsService.startCommand="npm start" generator.nodeJsService.runnerVersion=node

# edit serviceMaster, ubah mode='worker', port=3001 
vim zaruba-tasks/serviceWorker.zaruba.yaml

# bikin serviceAjaib dan moduleAjaib
zaruba please makeFastApiService generator.fastApi.service.name=serviceAjaib
zaruba please makeFastApiModule generator.fastApi.service.name=serviceAjaib generator.fastApi.module.name=moduleAjaib

# bikin route handler get /info
zaruba please makeFastApiRoute generator.fastApi.service.name=serviceAjaib generator.fastApi.module.name=moduleAjaib generator.fastApi.httpMethod=get generator.fastApi.url=/info

# bikin crud engineer
zaruba please makeFastApiCrud generator.fastApi.service.name=serviceAjaib generator.fastApi.module.name=moduleAjaib generator.fastApi.crud.entity=engineer generator.fastApi.crud.fields=name,role

# bikin runner untuk serviceAjaib
zaruba please makeFastApiServiceTask generator.service.location=serviceAjaib

# edit serviceAjaib, ubah port=3002, ubah juga connection string ke sqlite:///database.db
vim zaruba-tasks/serviceAjaib.zaruba.yaml

# run semuanya
zaruba please run

# tambah mysql
zaruba please makeMysqlDockerTask generator.service.name=mySqlAjaib

# edit serviceAjaib, tambahkan `runMySqlAjaib` sebagai dependency dari `runServiceAjaib`
vim zaruba-tasks/serviceAjaib.zaruba.yaml

# init helm
zaruba please initHelm

# bikin helm deployment untuk serviceAjaib
zaruba please makeHelmDeployment generator.service.name=serviceAjaib

# apply dan destroy helm deployment
zaruba please helmApply
zaruba please helmDestroy
```