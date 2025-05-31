---
layout: default
title: Day 15: Controller層の実装 - 解答例
---
# Day 15: Controller層の実装 - 解答例

## 課題1: TodoControllerの実装

```java
package com.example.todo.app.todo;

import java.util.List;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.core.annotation.AuthenticationPrincipal;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.validation.BindingResult;
import org.springframework.validation.annotation.Validated;
import org.springframework.web.bind.WebDataBinder;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.InitBinder;
import org.springframework.web.bind.annotation.ModelAttribute;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.servlet.mvc.support.RedirectAttributes;

import com.example.todo.app.common.BeanMapper;
import com.example.todo.domain.exception.ResourceNotFoundException;
import com.example.todo.domain.model.Todo;
import com.example.todo.domain.service.todo.TodoService;
import com.example.todo.domain.service.user.UserDetails;

@Controller
@RequestMapping("/todos")
public class TodoController {
    
    @Autowired
    private TodoService todoService;
    
    @Autowired
    private TodoValidator todoValidator;
    
    @Autowired
    private BeanMapper beanMapper;
    
    @ModelAttribute
    public TodoForm setUpForm() {
        return new TodoForm();
    }
    
    @InitBinder("todoForm")
    public void initBinder(WebDataBinder binder) {
        binder.addValidators(todoValidator);
    }
    
    @GetMapping
    public String list(@AuthenticationPrincipal UserDetails userDetails,
                      @RequestParam(required = false) Boolean completed,
                      Model model) {
        List<Todo> todos;
        if (completed != null) {
            todos = todoService.findByUserIdAndCompleted(userDetails.getUserId(), completed);
        } else {
            todos = todoService.findByUserId(userDetails.getUserId());
        }
        model.addAttribute("todos", todos);
        return "todo/list";
    }
    
    @GetMapping("/{id}")
    public String show(@AuthenticationPrincipal UserDetails userDetails,
                      @PathVariable Long id,
                      Model model) {
        Todo todo = todoService.findOne(id);
        if (todo == null || !todo.getUserId().equals(userDetails.getUserId())) {
            throw new ResourceNotFoundException("Todo not found");
        }
        model.addAttribute("todo", todo);
        return "todo/show";
    }
    
    @GetMapping("/create")
    public String createForm() {
        return "todo/createForm";
    }
    
    @PostMapping("/create")
    public String create(@AuthenticationPrincipal UserDetails userDetails,
                        @Validated TodoForm todoForm,
                        BindingResult bindingResult,
                        Model model,
                        RedirectAttributes redirectAttributes) {
        if (bindingResult.hasErrors()) {
            return "todo/createForm";
        }
        
        Todo todo = beanMapper.map(todoForm, Todo.class);
        todo.setUserId(userDetails.getUserId());
        todo.setCompleted(false);
        
        todoService.create(todo);
        
        redirectAttributes.addFlashAttribute("successMessage", "Todo created successfully");
        
        return "redirect:/todos";
    }
    
    @GetMapping("/{id}/edit")
    public String editForm(@AuthenticationPrincipal UserDetails userDetails,
                          @PathVariable Long id,
                          Model model) {
        Todo todo = todoService.findOne(id);
        if (todo == null || !todo.getUserId().equals(userDetails.getUserId())) {
            throw new ResourceNotFoundException("Todo not found");
        }
        
        TodoForm todoForm = beanMapper.map(todo, TodoForm.class);
        model.addAttribute("todoForm", todoForm);
        model.addAttribute("todoId", id);
        
        return "todo/editForm";
    }
    
    @PostMapping("/{id}/edit")
    public String edit(@AuthenticationPrincipal UserDetails userDetails,
                      @PathVariable Long id,
                      @Validated TodoForm todoForm,
                      BindingResult bindingResult,
                      Model model,
                      RedirectAttributes redirectAttributes) {
        if (bindingResult.hasErrors()) {
            model.addAttribute("todoId", id);
            return "todo/editForm";
        }
        
        Todo todo = todoService.findOne(id);
        if (todo == null || !todo.getUserId().equals(userDetails.getUserId())) {
            throw new ResourceNotFoundException("Todo not found");
        }
        
        beanMapper.map(todoForm, todo);
        
        todoService.update(todo);
        
        redirectAttributes.addFlashAttribute("successMessage", "Todo updated successfully");
        
        return "redirect:/todos";
    }
    
    @PostMapping("/{id}/delete")
    public String delete(@AuthenticationPrincipal UserDetails userDetails,
                        @PathVariable Long id,
                        RedirectAttributes redirectAttributes) {
        Todo todo = todoService.findOne(id);
        if (todo == null || !todo.getUserId().equals(userDetails.getUserId())) {
            throw new ResourceNotFoundException("Todo not found");
        }
        
        todoService.delete(id);
        
        redirectAttributes.addFlashAttribute("successMessage", "Todo deleted successfully");
        
        return "redirect:/todos";
    }
    
    @ExceptionHandler(ResourceNotFoundException.class)
    public String handleResourceNotFoundException(ResourceNotFoundException e, Model model) {
        model.addAttribute("errorMessage", e.getMessage());
        return "error/resourceNotFound";
    }
}
```

**TodoForm.java**

```java
package com.example.todo.app.todo;

import javax.validation.constraints.NotEmpty;
import javax.validation.constraints.Size;

import lombok.Data;

@Data
public class TodoForm {
    
    @NotEmpty
    @Size(max = 256)
    private String title;
    
    private String description;
    
    private boolean completed;
}
```

**TodoValidator.java**

```java
package com.example.todo.app.todo;

import org.springframework.stereotype.Component;
import org.springframework.validation.Errors;
import org.springframework.validation.Validator;

@Component
public class TodoValidator implements Validator {
    
    @Override
    public boolean supports(Class<?> clazz) {
        return TodoForm.class.isAssignableFrom(clazz);
    }
    
    @Override
    public void validate(Object target, Errors errors) {
        TodoForm form = (TodoForm) target;
        
        // タイトルが空白のみの場合
        if (form.getTitle() != null && form.getTitle().trim().isEmpty()) {
            errors.rejectValue("title", "todo.title.empty");
        }
    }
}
```

## 課題2: UserControllerの実装

```java
package com.example.todo.app.user;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.validation.BindingResult;
import org.springframework.validation.annotation.Validated;
import org.springframework.web.bind.WebDataBinder;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.InitBinder;
import org.springframework.web.bind.annotation.ModelAttribute;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.servlet.mvc.support.RedirectAttributes;

import com.example.todo.app.common.BeanMapper;
import com.example.todo.domain.model.User;
import com.example.todo.domain.service.user.UserService;

@Controller
@RequestMapping("/users")
public class UserController {
    
    @Autowired
    private UserService userService;
    
    @Autowired
    private UserValidator userValidator;
    
    @Autowired
    private BeanMapper beanMapper;
    
    @ModelAttribute
    public UserForm setUpForm() {
        return new UserForm();
    }
    
    @InitBinder("userForm")
    public void initBinder(WebDataBinder binder) {
        binder.addValidators(userValidator);
    }
    
    @GetMapping("/register")
    public String registerForm() {
        return "user/registerForm";
    }
    
    @PostMapping("/register")
    public String register(@Validated UserForm userForm,
                          BindingResult bindingResult,
                          Model model,
                          RedirectAttributes redirectAttributes) {
        if (bindingResult.hasErrors()) {
            return "user/registerForm";
        }
        
        if (userService.exists(userForm.getUsername())) {
            bindingResult.rejectValue("username", "user.exists");
            return "user/registerForm";
        }
        
        if (!userForm.getPassword().equals(userForm.getConfirmPassword())) {
            bindingResult.rejectValue("confirmPassword", "user.password.unmatch");
            return "user/registerForm";
        }
        
        User user = beanMapper.map(userForm, User.class);
        user.setEnabled(true);
        
        userService.create(user);
        
        redirectAttributes.addFlashAttribute("successMessage", "User registered successfully");
        
        return "redirect:/login";
    }
}
```

**UserForm.java**

```java
package com.example.todo.app.user;

import javax.validation.constraints.NotEmpty;
import javax.validation.constraints.Size;

import lombok.Data;

@Data
public class UserForm {
    
    @NotEmpty
    @Size(min = 4, max = 100)
    private String username;
    
    @NotEmpty
    @Size(min = 8, max = 100)
    private String password;
    
    @NotEmpty
    private String confirmPassword;
}
```

**UserValidator.java**

```java
package com.example.todo.app.user;

import org.springframework.stereotype.Component;
import org.springframework.validation.Errors;
import org.springframework.validation.Validator;

@Component
public class UserValidator implements Validator {
    
    @Override
    public boolean supports(Class<?> clazz) {
        return UserForm.class.isAssignableFrom(clazz);
    }
    
    @Override
    public void validate(Object target, Errors errors) {
        UserForm form = (UserForm) target;
        
        // ユーザー名が空白のみの場合
        if (form.getUsername() != null && form.getUsername().trim().isEmpty()) {
            errors.rejectValue("username", "user.username.empty");
        }
        
        // パスワードが空白のみの場合
        if (form.getPassword() != null && form.getPassword().trim().isEmpty()) {
            errors.rejectValue("password", "user.password.empty");
        }
        
        // パスワードが一致しない場合
        if (form.getPassword() != null && form.getConfirmPassword() != null
                && !form.getPassword().equals(form.getConfirmPassword())) {
            errors.rejectValue("confirmPassword", "user.password.unmatch");
        }
    }
}
```

## 課題3: 例外ハンドリングの実装

```java
package com.example.todo.app.common;

import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseStatus;
import org.springframework.web.servlet.ModelAndView;

import com.example.todo.domain.exception.ResourceNotFoundException;

@ControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(ResourceNotFoundException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public ModelAndView handleResourceNotFoundException(ResourceNotFoundException e) {
        ModelAndView modelAndView = new ModelAndView("error/resourceNotFound");
        modelAndView.addObject("errorMessage", e.getMessage());
        return modelAndView;
    }
    
    @ExceptionHandler(Exception.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public ModelAndView handleException(Exception e) {
        ModelAndView modelAndView = new ModelAndView("error/systemError");
        modelAndView.addObject("errorMessage", "An error occurred");
        return modelAndView;
    }
}
```

**ResourceNotFoundException.java**

```java
package com.example.todo.domain.exception;

public class ResourceNotFoundException extends RuntimeException {
    
    private static final long serialVersionUID = 1L;
    
    public ResourceNotFoundException(String message) {
        super(message);
    }
}
```

## 課題4: コントローラのテスト実装

```java
package com.example.todo.app.todo;

import static org.mockito.Mockito.*;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

import java.util.Arrays;
import java.util.List;

import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.MockitoAnnotations;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.setup.MockMvcBuilders;

import com.example.todo.app.common.BeanMapper;
import com.example.todo.domain.exception.ResourceNotFoundException;
import com.example.todo.domain.model.Todo;
import com.example.todo.domain.service.todo.TodoService;
import com.example.todo.domain.service.user.UserDetails;

@RunWith(SpringRunner.class)
public class TodoControllerTest {
    
    @Mock
    private TodoService todoService;
    
    @Mock
    private TodoValidator todoValidator;
    
    @Mock
    private BeanMapper beanMapper;
    
    @Mock
    private Authentication authentication;
    
    @InjectMocks
    private TodoController todoController;
    
    private MockMvc mockMvc;
    
    private UserDetails userDetails;
    
    @Before
    public void setUp() {
        MockitoAnnotations.initMocks(this);
        
        userDetails = new UserDetails();
        userDetails.setUserId(1L);
        userDetails.setUsername("user1");
        
        when(authentication.getPrincipal()).thenReturn(userDetails);
        SecurityContextHolder.getContext().setAuthentication(authentication);
        
        mockMvc = MockMvcBuilders.standaloneSetup(todoController)
                .setControllerAdvice(new GlobalExceptionHandler())
                .build();
    }
    
    @Test
    public void testList() throws Exception {
        // 準備
        Todo todo1 = new Todo();
        todo1.setId(1L);
        todo1.setTitle("Todo 1");
        todo1.setUserId(1L);
        
        Todo todo2 = new Todo();
        todo2.setId(2L);
        todo2.setTitle("Todo 2");
        todo2.setUserId(1L);
        
        List<Todo> todos = Arrays.asList(todo1, todo2);
        
        when(todoService.findByUserId(1L)).thenReturn(todos);
        
        // 実行と検証
        mockMvc.perform(get("/todos"))
                .andExpect(status().isOk())
                .andExpect(view().name("todo/list"))
                .andExpect(model().attribute("todos", todos));
    }
    
    @Test
    public void testShow() throws Exception {
        // 準備
        Todo todo = new Todo();
        todo.setId(1L);
        todo.setTitle("Todo 1");
        todo.setUserId(1L);
        
        when(todoService.findOne(1L)).thenReturn(todo);
        
        // 実行と検証
        mockMvc.perform(get("/todos/1"))
                .andExpect(status().isOk())
                .andExpect(view().name("todo/show"))
                .andExpect(model().attribute("todo", todo));
    }
    
    @Test
    public void testShowNotFound() throws Exception {
        // 準備
        when(todoService.findOne(1L)).thenReturn(null);
        
        // 実行と検証
        mockMvc.perform(get("/todos/1"))
                .andExpect(status().isNotFound())
                .andExpect(view().name("error/resourceNotFound"));
    }
    
    @Test
    public void testCreateForm() throws Exception {
        // 実行と検証
        mockMvc.perform(get("/todos/create"))
                .andExpect(status().isOk())
                .andExpect(view().name("todo/createForm"))
                .andExpect(model().attributeExists("todoForm"));
    }
    
    @Test
    public void testCreate() throws Exception {
        // 準備
        TodoForm todoForm = new TodoForm();
        todoForm.setTitle("New Todo");
        todoForm.setDescription("Description");
        
        Todo todo = new Todo();
        todo.setTitle("New Todo");
        todo.setDescription("Description");
        
        when(beanMapper.map(any(TodoForm.class), eq(Todo.class))).thenReturn(todo);
        
        // 実行と検証
        mockMvc.perform(post("/todos/create")
                .param("title", "New Todo")
                .param("description", "Description"))
                .andExpect(status().is3xxRedirection())
                .andExpect(redirectedUrl("/todos"))
                .andExpect(flash().attributeExists("successMessage"));
        
        verify(todoService).create(todo);
    }
    
    @Test
    public void testCreateValidationError() throws Exception {
        // 準備
        doAnswer(invocation -> {
            Errors errors = invocation.getArgument(1);
            errors.rejectValue("title", "todo.title.empty");
            return null;
        }).when(todoValidator).validate(any(), any());
        
        // 実行と検証
        mockMvc.perform(post("/todos/create")
                .param("title", "")
                .param("description", "Description"))
                .andExpect(status().isOk())
                .andExpect(view().name("todo/createForm"))
                .andExpect(model().attributeHasFieldErrors("todoForm", "title"));
        
        verify(todoService, never()).create(any());
    }
    
    @Test
    public void testEditForm() throws Exception {
        // 準備
        Todo todo = new Todo();
        todo.setId(1L);
        todo.setTitle("Todo 1");
        todo.setDescription("Description");
        todo.setUserId(1L);
        
        TodoForm todoForm = new TodoForm();
        todoForm.setTitle("Todo 1");
        todoForm.setDescription("Description");
        
        when(todoService.findOne(1L)).thenReturn(todo);
        when(beanMapper.map(todo, TodoForm.class)).thenReturn(todoForm);
        
        // 実行と検証
        mockMvc.perform(get("/todos/1/edit"))
                .andExpect(status().isOk())
                .andExpect(view().name("todo/editForm"))
                .andExpect(model().attribute("todoForm", todoForm))
                .andExpect(model().attribute("todoId", 1L));
    }
    
    @Test
    public void testEdit() throws Exception {
        // 準備
        Todo todo = new Todo();
        todo.setId(1L);
        todo.setTitle("Todo 1");
        todo.setDescription("Description");
        todo.setUserId(1L);
        
        when(todoService.findOne(1L)).thenReturn(todo);
        
        // 実行と検証
        mockMvc.perform(post("/todos/1/edit")
                .param("title", "Updated Todo")
                .param("description", "Updated Description"))
                .andExpect(status().is3xxRedirection())
                .andExpect(redirectedUrl("/todos"))
                .andExpect(flash().attributeExists("successMessage"));
        
        verify(todoService).update(todo);
    }
    
    @Test
    public void testDelete() throws Exception {
        // 準備
        Todo todo = new Todo();
        todo.setId(1L);
        todo.setTitle("Todo 1");
        todo.setUserId(1L);
        
        when(todoService.findOne(1L)).thenReturn(todo);
        
        // 実行と検証
        mockMvc.perform(post("/todos/1/delete"))
                .andExpect(status().is3xxRedirection())
                .andExpect(redirectedUrl("/todos"))
                .andExpect(flash().attributeExists("successMessage"));
        
        verify(todoService).delete(1L);
    }
}
```

### 解説

課題1～4の解答例では、TERASOLUNAフレームワークの推奨パターンに基づいて、Controller層の実装を行いました。

1. **TodoControllerの実装**
   - リソース指向の設計（/todosをベースとしたURL設計）
   - PRGパターンの適用（フォーム送信後のリダイレクト）
   - フラッシュスコープの活用（リダイレクト後のメッセージ表示）
   - 例外ハンドリング（ResourceNotFoundException）
   - セキュリティの考慮（ユーザーIDによるアクセス制御）

2. **UserControllerの実装**
   - ユーザー登録機能の実装
   - バリデーションの適用（@Validated、BindingResult）
   - カスタムバリデーションの実装（UserValidator）
   - ビジネスルールの適用（ユーザー名の重複チェック、パスワードの一致チェック）

3. **例外ハンドリングの実装**
   - グローバルな例外ハンドリング（@ControllerAdvice）
   - 特定の例外に対する処理（ResourceNotFoundException）
   - 汎用的な例外処理（Exception）
   - HTTPステータスコードの設定（@ResponseStatus）

4. **コントローラのテスト実装**
   - MockMvcを使用したテスト
   - モックオブジェクトの活用（@Mock、@InjectMocks）
   - セキュリティコンテキストの設定
   - 正常系・異常系のテスト
   - バリデーションエラーのテスト

これらの実装は、実務でのWebアプリケーション開発において、保守性と拡張性の高いController層を構築するための基礎となります。特に、PRGパターンやフラッシュスコープの活用、適切な例外ハンドリングは、ユーザーエクスペリエンスの向上に寄与します。また、テストの実装により、コードの品質と信頼性を確保することができます。
