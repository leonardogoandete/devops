### SideCar

O padra SideCar e utilizado no Kubernetes para adicionar funcionalidades a um container principal. O SideCar é um container que roda junto com o container principal e pode ser utilizado para adicionar funcionalidades como logs, monitoramento, etc.

### Adapter

O padrao Adapter é utilizado para adaptar um container a um serviço. Por exemplo, se um container precisa acessar um serviço que não é compatível com o formato de dados que ele espera, um container Adapter pode ser utilizado para adaptar o formato de dados.

Por exemplo: Temos um conjunto de dispositivos IoT que enviam dados em formato JSON para um serviço. Se um container precisa acessar esse serviço, mas espera receber os dados em formato XML, um container Adapter pode ser utilizado para converter os dados de JSON para XML.

### Ambassador

O padrao Ambassador é utilizado para intermediar a comunicação entre um container e um serviço. Por exemplo, se um container precisa acessar um serviço que não é compatível com o formato de dados que ele espera, um container Ambassador pode ser utilizado para intermediar a comunicação entre o container e o serviço.

Para o container a comunicação é transparente, ele não sabe que está se comunicando com um container Ambassador, pois para ele parece uma comunicação local.

### Controllers

O padrao Controllers é utilizado para controlar o comportamento de um container. Por exemplo, se um container precisa ser reiniciado quando um arquivo é modificado, um container Controller pode ser utilizado para monitorar o arquivo e reiniciar o container quando o arquivo é modificado. Ou seja podemos criar nossos próprios controllers para monitorar e controlar o comportamento de um container.

### Operators

O padrao Operators é utilizado para automatizar tarefas de gerenciamento de um container. Por exemplo, se um container precisa ser reiniciado quando um arquivo é modificado, um container Operator pode ser utilizado para monitorar o arquivo e reiniciar o container quando o arquivo é modificado. Ou seja podemos criar nossos próprios operators para automatizar tarefas de gerenciamento de um container.