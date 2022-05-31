## **Visão Geral**
O **dotnet-bucket-plugin** adiciona em uma stack a capacidade de provisionar o uso do Amazon Simple Storage Service (S3) seja recuperando, salvando ou apagando objetos.

## **Uso**

#### **Pré-requisitos**
Para utilizar esse plugin é necessário ter uma stack dotnet criada pelo `CLI` do `StackSpot` que você pode baixar [**aqui**](https://stackspot.com/).

Ter instalado:
- .NET 5 ou 6 
- O template `dotnet-api-template` ou o `dotnet-worker-template` deverá estar aplicado para você conseguir utilizar este plugin.

#### **Inputs**

* RegionEndpoint - Endpoint regional que será utilizado para requisitar o S3 - Campo Obrigatório.

Você pode sobrescrever a configuração padrão adicionando a seção `S3` em seu `appsettings.json`. Os valores aceitáveis você pode encontrar [aqui](https://docs.aws.amazon.com/pt_br/pt_br/AWSEC2/latest/WindowsGuide/using-regions-availability-zones.html#concepts-available-regions).

```json
  "S3": {
      "RegionEndpoint": "sa-east-1"
  }
```

Adicione ao seu `IServiceCollection` via `services.AddBucketS3()` no `Startup` da aplicação ou `Program` tendo como parêmetro de entrada `IConfiguration` e `IWebHostEnvironment`. 

```csharp
services.AddBucketS3(Configuration);
```

#### **Operações**

*  DeleteAsync - Remove a versão nula(se houver) de um objeto e insere um marcador de exclusão, tornando a versão mais recente do objeto. Em caso de não haver uma versão nula, nenum objeto é removido.

```csharp
public class Service
{
    private readonly IBucketS3 _bucketS3;

    public Service(IBucketS3 bucketS3)
    {
        _bucketS3 = bucketS3;
    }
    
    public async Task DeleteAsync()
    {
        var bucket = new Common.Bucket()
        {
            BucketName = "bucket-test",
            Key = "bucket-key"
        };
        await _bucketS3.DeleteAsync(bucket);
    }
}
```

* GetStreamAsync - Recupera um objeto.

```csharp
public class Service
{
    private readonly IBucketS3 _bucketS3;

    public Service(IBucketS3 bucketS3)
    {
        _bucketS3 = bucketS3;
    }
    
    public async Task GetStreamAsync()
    {
        var bucket = new Common.Bucket()
        {
            BucketName = "bucket-test",
            Key = "bucket-key"
        };
        var stream = await _bucketS3.GetStreamAsync(bucket);
    }

    public async Task GetGenericStreamAsync()
    {
        var bucket = new Common.Bucket()
        {
            BucketName = "bucket-test",
            Key = "bucket-key"
        };
        Example example = await _bucketS3.GetStreamAsync<Example>(bucket, ContentType.Json);
    }
}
```

* UploadAsync - Adiciona um objeto ao bucket.

```csharp
public class Service
{
    private readonly IBucketS3 _bucketS3;

    public Service(IBucketS3 bucketS3)
    {
        _bucketS3 = bucketS3;
    }
    
    public async Task UploadAsync(Example example)
    {
        var bucket = new Common.Bucket()
        {
            BucketName = "bucket-test",
            Key = "bucket-key"
        };

        var str = JsonConvert.SerializeObject(example);
        byte[] byteArray = Encoding.UTF8.GetBytes(str);

        await _bucketS3.UploadAsync(bucket, byteArray, ContentType.Json);
    }

    public async Task UploadAsync(Example example)
    {
        var bucket = new Common.Bucket()
        {
            BucketName = "bucket-test",
            Key = "bucket-key"
        };

        var str = JsonConvert.SerializeObject(example);

        await _bucketS3.UploadAsync(bucket, str, ContentType.Json);
    }
}
```


#### 4. Ambiente local

* Esta etapa não é obrigatória.
* Recomendamos, para o desenvolvimento local, a criação de um contâiner com a imagem do [Localstack](https://github.com/localstack/localstack). 
* Para o funcionamento local você deve preencher a variável de ambiente `LOCALSTACK_CUSTOM_SERVICE_URL` com o valor da url do serviço. O valor padrão do localstack é http://localhost:4566.
* Abaixo um exemplo de arquivo `docker-compose` com a criação do contâiner: 

```yaml
version: '2.1'

services:
  localstack:
    image: localstack/localstack
    ports:
      - "4566:4566"
    environment:
      - SERVICES=s3
      - AWS_DEFAULT_OUTPUT=json
      - DEFAULT_REGION=sa-east-1
```

Após a criação do contâiner, crie um bucket para realizar os testes com o componente. Recomendamos que você tenha instalado em sua estação o [AWS CLI](https://aws.amazon.com/pt/cli/). Abaixo um exemplo de comando para criação de um bucket:

```bash
aws --endpoint-url=http://localhost:4566  --region=sa-east-1 s3 mb s3://[NOME DO BUCKET]
```

### **Implementação**
- [**Nuget**](https://www.nuget.org/packages/StackSpot.Bucket.S3/)