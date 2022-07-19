## **Visão Geral**
O plugin **`dotnet-bucket-plugin`** adiciona em uma Stack a capacidade de provisionar o uso do **Amazon Simple Storage Service** (S3), seja recuperando, salvando ou apagando objetos.

## **Uso**

#### **Pré-requisitos**
Para utilizar este plugin é preciso ter instalado na sua máquina os itens abaixo:

- Uma Stack dotNet criada pelo [**STK CLI**](https://stackspot.com/);  
- .NET 5 ou 6;   
- O template **`dotnet-api-template`** ou o **`dotnet-worker-template`** já aplicados. 

### **Inputs configurados automaticamente**  

As etapas abaixo são configuradas automaticamente pelo plugin.   

- #### Aplicação do `RegionEndpoint`  
O endpoint regional será utilizado para requisitar o **`S3`**. 

```json
  "S3": {
      "RegionEndpoint": "sa-east-1"
  }
```
> É possível sobrescrever a configuração padrão adicionando a seção **`S3`** no seu **`appsettings.json`**. Os valores aceitáveis são encontrados [aqui](https://docs.aws.amazon.com/pt_br/pt_br/AWSEC2/latest/WindowsGuide/using-regions-availability-zones.html#concepts-available-regions).

- #### Configuração do `IServiceCollection`

A configuração abaixo será feita no **`IServiceCollection`**, através do `services.AddBucketS3()`, no `Startup` da aplicação ou do `Program`. Ela terá **`IConfiguration`** e **`IWebHostEnvironment`** como parâmetros de entrada. 

```csharp
services.AddBucketS3(Configuration);
```

### **Exemplos de aplicação do plugin**

Confira abaixo alguns exemplos de aplicação do **`dotnet-bucket-plugin`**:  

- #### DeleteAsync  
Se houver uma versão nula de um objeto, ela é removida e um marcador de exlusão é inserido. Isso atualiza a versão do objeto. Caso não haja uma versão nula, nenhum objeto é removido ou atualizado. 

Confira a execução abaixo:  

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

- #### GetStreamAsync  
É responsável por recuperar um objeto.

Confira a execução abaixo:  

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

- #### UploadAsync  
Adiciona um objeto ao bucket.

Confira a execução abaixo:  

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

- #### Execução em ambiente local  

Para fazer o desenvolvimento local do plugin, crie um contêiner com a imagem do [**LocalStack**](https://github.com/localstack/localstack). 

Para que funcione localmente, preencha a variável de ambiente **`LOCALSTACK_CUSTOM_SERVICE_URL`** com o valor da URL do serviço. O valor padrão do LocalStack é **http://localhost:4566**.

Confira abaixo um exemplo de arquivo **`docker-compose`** com a criação do contêiner: 

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

Depois de criar o contêiner, crie um bucket para fazer os testes com o componente. 

É recomendado que você tenha instalado em sua estação o [**AWS CLI**](https://aws.amazon.com/pt/cli/). 

Confira abaixo um exemplo de comando para criar um bucket:

```bash
aws --endpoint-url=http://localhost:4566  --region=sa-east-1 s3 mb s3://[NOME DO BUCKET]
```
