# Regras de Desenvolvimento de APIs REST Java (com.studygenie.lesson)

Sempre siga as seguintes diretrizes ao criar ou modificar APIs REST Java neste projeto:

## 1. Localização e Estrutura de Arquivos
- **Controller**: Deve ficar sempre no diretório/pacote `controller` em:
  `com.studygenie.lesson.controller` (ex: `TagController.java`).
- **Arquivos de Suporte da Entidade**: Deve ser criada uma pasta (pacote) em letras minúsculas com o mesmo nome da entidade dentro de `com.studygenie.lesson` para todos os outros arquivos da entidade:
  - Modelo de Entidade (ex: `Tag.java`)
  - Repositório (ex: `TagRepository.java`)
  - Request DTO (ex: `TagRequestDTO.java`)
  - Response DTO (ex: `TagResponseDTO.java`)
  
  *Exemplo de estrutura para a entidade `Tag`:*
  - `com/studygenie/lesson/controller/TagController.java`
  - `com/studygenie/lesson/tag/Tag.java`
  - `com/studygenie/lesson/tag/TagRepository.java`
  - `com/studygenie/lesson/tag/TagRequestDTO.java`
  - `com/studygenie/lesson/tag/TagResponseDTO.java`

## 2. Mapeamento de Entidades e Banco de Dados
- O nome da tabela e da entidade JPA deve ser sempre no **plural** (ex: `@Table(name = "tags")` e `@Entity(name = "tags")`).
- Para os identificadores únicos (IDs), use sempre o tipo de dado `UUID` com geração automática:
  ```java
  @Id
  @GeneratedValue(strategy = GenerationType.UUID)
  private UUID id;
  ```

## 3. Data Transfer Objects (DTOs)
- Utilize registros (`record`) para definir `RequestDTO` e `ResponseDTO`.
- **Validação de Payload**: No Request DTO, crie validações utilizando anotações do Jakarta Validation (como `@NotBlank`, `@NotNull`, `@Size`, etc.) para garantir que o payload venha com as informações válidas.
- O DTO de Request e Response deve ser o único responsável por realizar o mapeamento entre a entidade e o DTO (e vice-versa):
  - No DTO de Request, inclua métodos como:
    - `public Entidade toEntidade() { ... }`
    - `public void mapToEntidade(Entidade entidade) { ... }`
  - No DTO de Response, inclua um método estático de fábrica:
    - `public static EntidadeResponseDTO from(Entidade entidade) { ... }`
- **Mapeamento Estrito**: Apenas os campos explícitos do DTO devem ser mapeados/carregados. Quaisquer outros dados da entidade que não estejam no DTO não devem ser carregados pelo DTO.

## 4. Controladores e Endpoints
- O nome principal do endpoint (definido no `@RequestMapping` do controller) deve ser sempre no **plural** (ex: `@RequestMapping("/tags")`).
- Use sempre `ResponseEntity` para retornar as respostas de todos os endpoints do controller (ex: `ResponseEntity<TagResponseDTO>`, `ResponseEntity<Void>`, `ResponseEntity.ok(...)`).
- A injeção de dependência deve ser feita sempre utilizando a anotação `@Autowired` nos campos/atributos.

---

### Exemplo de Referência: Entidade `Tag`

#### Controller:
```java
package com.studygenie.lesson.controller;

import java.util.UUID;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import com.studygenie.lesson.tag.*;

@RestController
@RequestMapping("/tags")
public class TagController {
    @Autowired
    private TagRepository tagRepository;

    @GetMapping("/{id}")
    public ResponseEntity<TagResponseDTO> get(@PathVariable UUID id) {
        return tagRepository.findById(id)
                .map(tag -> ResponseEntity.ok(TagResponseDTO.from(tag)))
                .orElse(ResponseEntity.notFound().build());
    }
}
```

#### Entidade:
```java
package com.studygenie.lesson.tag;

import java.util.UUID;
import jakarta.persistence.*;
import lombok.Data;

@Table(name = "tags")
@Entity(name = "tags")
@Data
public class Tag {
    @Id
    @GeneratedValue(strategy = GenerationType.UUID)
    private UUID id;
    private String name;
}
```

#### Request DTO:
```java
package com.studygenie.lesson.tag;

import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.Size;

public record TagRequestDTO(
    @NotBlank(message = "Tag name is required") 
    @Size(max = 20, message = "Tag name must be less than 20 characters") 
    String name
) {
    public Tag toTag() {
        Tag tag = new Tag();
        tag.setName(this.name);
        return tag;
    }
    public void mapToTag(Tag tag) {
        tag.setName(this.name);
    }
}
```

#### Response DTO:
```java
package com.studygenie.lesson.tag;

import java.util.UUID;

public record TagResponseDTO(UUID id, String name) {
    public static TagResponseDTO from(Tag tag) {
        return new TagResponseDTO(tag.getId(), tag.getName());
    }
}
```
