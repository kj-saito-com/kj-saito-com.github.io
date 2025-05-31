---
layout: default
title: "Day 19: Form/DTOの実装 - 解答例"
---
# Day 19: Form/DTOの実装 - 解答例

## 課題1: TodoFormの実装

**TodoForm.java**

```java
package com.example.todo.app.todo;

import java.io.Serializable;

import javax.validation.constraints.NotBlank;
import javax.validation.constraints.Size;

import com.example.todo.domain.model.Todo;

import lombok.Data;

@Data
public class TodoForm implements Serializable {
    
    private static final long serialVersionUID = 1L;
    
    @NotBlank(message = "Title is required.")
    @Size(max = 100, message = "Title must be less than or equal to {max} characters.")
    private String title;
    
    @Size(max = 1000, message = "Description must be less than or equal to {max} characters.")
    private String description;
    
    private boolean completed;
    
    /**
     * TodoFormからTodoエンティティを作成します。
     * 
     * @param userId ユーザーID
     * @return Todoエンティティ
     */
    public Todo toEntity(Long userId) {
        Todo todo = new Todo();
        todo.setTitle(title);
        todo.setDescription(description);
        todo.setCompleted(completed);
        todo.setUserId(userId);
        return todo;
    }
    
    /**
     * TodoFormからTodoエンティティを更新します。
     * 
     * @param todo 更新対象のTodoエンティティ
     * @return 更新されたTodoエンティティ
     */
    public Todo updateEntity(Todo todo) {
        todo.setTitle(title);
        todo.setDescription(description);
        if (completed && !todo.isCompleted()) {
            todo.complete();
        } else if (!completed && todo.isCompleted()) {
            todo.incomplete();
        }
        return todo;
    }
    
    /**
     * Todoエンティティの値をTodoFormに設定します。
     * 
     * @param todo Todoエンティティ
     * @return TodoForm
     */
    public static TodoForm fromEntity(Todo todo) {
        TodoForm form = new TodoForm();
        form.setTitle(todo.getTitle());
        form.setDescription(todo.getDescription());
        form.setCompleted(todo.isCompleted());
        return form;
    }
}
```

## 課題2: TodoSearchFormの実装

**TodoSearchForm.java**

```java
package com.example.todo.app.todo;

import java.io.Serializable;

import com.example.todo.domain.repository.todo.TodoSearchCriteria;

import lombok.Data;

@Data
public class TodoSearchForm implements Serializable {
    
    private static final long serialVersionUID = 1L;
    
    private String title;
    
    private Boolean completed;
    
    private String orderBy;
    
    private Boolean ascending;
    
    /**
     * TodoSearchFormからTodoSearchCriteriaを作成します。
     * 
     * @param userId ユーザーID
     * @return TodoSearchCriteria
     */
    public TodoSearchCriteria toSearchCriteria(Long userId) {
        TodoSearchCriteria criteria = new TodoSearchCriteria();
        criteria.setUserId(userId);
        criteria.setTitle(title);
        criteria.setCompleted(completed);
        criteria.setOrderBy(orderBy);
        criteria.setAscending(ascending);
        return criteria;
    }
}
```

## 課題3: TodoDtoの実装

**TodoDto.java**

```java
package com.example.todo.app.todo;

import java.io.Serializable;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;

import com.example.todo.domain.model.Todo;

import lombok.Data;

@Data
public class TodoDto implements Serializable {
    
    private static final long serialVersionUID = 1L;
    
    private Long id;
    
    private String title;
    
    private String description;
    
    private boolean completed;
    
    private String createdAt;
    
    private String updatedAt;
    
    private String completedAt;
    
    /**
     * Todoエンティティの値をTodoDtoに設定します。
     * 
     * @param todo Todoエンティティ
     * @return TodoDto
     */
    public static TodoDto fromEntity(Todo todo) {
        TodoDto dto = new TodoDto();
        dto.setId(todo.getId());
        dto.setTitle(todo.getTitle());
        dto.setDescription(todo.getDescription());
        dto.setCompleted(todo.isCompleted());
        
        DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
        
        if (todo.getCreatedAt() != null) {
            dto.setCreatedAt(todo.getCreatedAt().format(formatter));
        }
        
        if (todo.getUpdatedAt() != null) {
            dto.setUpdatedAt(todo.getUpdatedAt().format(formatter));
        }
        
        if (todo.getCompletedAt() != null) {
            dto.setCompletedAt(todo.getCompletedAt().format(formatter));
        }
        
        return dto;
    }
}
```

## 課題4: カスタム検証の実装

**UniqueTodoTitle.java**

```java
package com.example.todo.app.common.validation;

import static java.lang.annotation.ElementType.ANNOTATION_TYPE;
import static java.lang.annotation.ElementType.CONSTRUCTOR;
import static java.lang.annotation.ElementType.FIELD;
import static java.lang.annotation.ElementType.METHOD;
import static java.lang.annotation.ElementType.PARAMETER;
import static java.lang.annotation.ElementType.TYPE_USE;
import static java.lang.annotation.RetentionPolicy.RUNTIME;

import java.lang.annotation.Documented;
import java.lang.annotation.Retention;
import java.lang.annotation.Target;

import javax.validation.Constraint;
import javax.validation.Payload;

@Target({ METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER, TYPE_USE })
@Retention(RUNTIME)
@Documented
@Constraint(validatedBy = { UniqueTodoTitleValidator.class })
public @interface UniqueTodoTitle {
    
    String message() default "{com.example.todo.app.common.validation.UniqueTodoTitle.message}";
    
    Class<?>[] groups() default {};
    
    Class<? extends Payload>[] payload() default {};
    
    long userId() default 0;
    
    long todoId() default 0;
    
    @Target({ METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER, TYPE_USE })
    @Retention(RUNTIME)
    @Documented
    @interface List {
        UniqueTodoTitle[] value();
    }
}
```

**UniqueTodoTitleValidator.java**

```java
package com.example.todo.app.common.validation;

import javax.validation.ConstraintValidator;
import javax.validation.ConstraintValidatorContext;

import org.springframework.beans.factory.annotation.Autowired;

import com.example.todo.domain.service.TodoDomainService;

public class UniqueTodoTitleValidator implements ConstraintValidator<UniqueTodoTitle, String> {
    
    @Autowired
    private TodoDomainService todoDomainService;
    
    private long userId;
    
    private long todoId;
    
    @Override
    public void initialize(UniqueTodoTitle constraintAnnotation) {
        this.userId = constraintAnnotation.userId();
        this.todoId = constraintAnnotation.todoId();
    }
    
    @Override
    public boolean isValid(String value, ConstraintValidatorContext context) {
        if (value == null) {
            return true;
        }
        
        if (todoId == 0) {
            // 新規作成の場合
            return !todoDomainService.existsByUserIdAndTitle(userId, value);
        } else {
            // 更新の場合
            return !todoDomainService.existsByUserIdAndTitleAndIdNot(userId, value, todoId);
        }
    }
}
```

**ValidationMessages.properties**

```properties
com.example.todo.app.common.validation.UniqueTodoTitle.message=Todo with the same title already exists.
```

### 解説

課題1～4の解答例では、TERASOLUNAフレームワークの推奨パターンに基づいて、Form/DTOの実装を行いました。

1. **TodoFormの実装**
   - Bean Validationによる入力検証
   - ドメインオブジェクトとの変換メソッドの実装
   - 完了状態の変更処理の実装

2. **TodoSearchFormの実装**
   - 検索条件の保持
   - 検索条件の変換メソッドの実装
   - シリアライズ可能な設計

3. **TodoDtoの実装**
   - 画面表示用のデータ構造の提供
   - 日時の文字列変換
   - ドメインオブジェクトからの変換メソッドの実装

4. **カスタム検証の実装**
   - カスタムアノテーションの定義
   - 検証ロジックの実装
   - エラーメッセージの定義

これらの実装により、ユーザーインターフェース層とドメイン層の適切な分離、入力検証の実装、データ変換の手法などを実現しています。特に、ドメインオブジェクトとの変換メソッドを実装することで、レイヤー間のデータ受け渡しを容易にし、各層の責務を明確に分離しています。また、カスタム検証を実装することで、ビジネスルールに基づく検証を宣言的に行うことができます。

TodoFormの実装では、完了状態の変更処理を適切に実装することで、ドメインオブジェクトの振る舞いを活用しています。TodoDtoの実装では、日時の文字列変換を行うことで、画面表示用のデータ構造を提供しています。これらの実装は、実務でのアプリケーション開発において、ユーザーインターフェース層とドメイン層の適切な分離を実現するための重要な要素となります。
