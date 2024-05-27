---
# try also 'default' to start simple
theme: default
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
# background: https://cover.sli.dev
# some information about your slides, markdown enabled
title: Arquitetura Front-end orientada a Dados
# info: 
# apply any unocss classes to the current slide
# class: text-center
# https://sli.dev/custom/highlighters.html
# highlighter: shiki
# https://sli.dev/guide/drawing
drawings:
  persist: false
# slide transition: https://sli.dev/guide/animations#slide-transitions
transition: slide-left
# enable MDC Syntax: https://sli.dev/guide/syntax#mdc-syntax
mdc: true
colorSchema: dark
aspectRatio: 16/9
themeConfig:
  primary: '#5d8392'
plantUmlServer: 'https://www.plantuml.com/plantuml'

layout: cover
---

# Arquitetura Front-end orientada a Dados
## Trabalhando com CRUDs


---

<div class="flex gap-5 flex-col items-start">


<div class=" p-5 flex-1">

<div class="bg-[#1f1f1fcc] p-2 px-5 rounded flex gap-5 flex-row items-start">
<div class="flex flex-row gap-2 items-center justify-between">
  <span><img src="/images/paulo.jpeg" class="w-20 rounded-full" /></span>
  <div class="flex flex-col">
    Paulo Cândido
    <span class="text-sm">
      @paulhenrique
    </span>
  </div>
  <img src="/images/qr_code.png" class="w-15">
</div>
</div>

```json
{ 
  "name": "Paulo Cândido",
  "professional": [
    "Senior Frontend Develiper no CPQD",
    "Organizador do Front in Campinas",
    "Membro da Liga voluntária do MWPT"
  ],
  "technologies": [
    "React", "Vue", "Typescript", "3D Maps", "Progressive Web Apps"
  ],
  "likes": [
    "Animes", "Séries", "The Office", "Musician"
  ]
}
```
</div>

</div>

---
hide: false
transition: slide-up
---

# Agenda
- Introdução
- Como resolvemos problemas normalmente
- Como resolver problemas utilizando orientação aos dados 
- Como sumarizar isso e escalar 
- Show me the code

---
layout: image-right
image: /images/crud01.jpg
backgroundSize: contain
---

# Introdução

Por muitas, vezes somos levados a construir diversos tipos de formulários e cadastros, mas no fim, na verdade o que mais fazemos é o famigerado **CRUD**.

---
layout: intro
transition: slide-up
---

# Faz sentido ficar reinventando a roda toda vez que vamos trabalhar com um CRUD?

---
layout: image
image: /images/crud_meme_02.png
transition: slide-up
---

---
layout: center
transition: slide-up
---

# Como resolvemos isso normalmente?

---
transition: slide-up
---

# Primeiro a estrutura da página
Sim, vamos utilizar React nos exemplos

```tsx{*|10-17|10-17,4|5-7|*}
import React from "react";

export const UserRegister = () => {
  const [userName, setUserName] = React.useState<string>("");
  const handleSubmit: React.FormEventHandler<HTMLFormElement> = (e) => {
    e.preventDefault();
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="username"
        value={userName}
        onChange={(e) => setUserName(e.target.value)}
      />
      <button type="submit">Submit</button>
    </form>
  );
};

export default UserRegister;

```

---

E quando aumentamos a quantidade de itens no formulário?
```tsx{*|4-7|4-7,15-21|}{maxHeight: '50vh'}
import React from "react";

export const UserRegister = () => {
  const [userName, setUserName] = React.useState<string>("");
  const [email, setEmail] = React.useState<string>("");
  const [name, setName] = React.useState<string>("");
  const [password, setPassword] = React.useState<string>("");
  
  const handleSubmit: React.FormEventHandler<HTMLFormElement> = (e) => {
    e.preventDefault();
    submitForm({ userName, email, name, password })
  };

  return (
    <form onSubmit={handleSubmit}>
      <input type="name" value={name} onChange={(e) => setName(e.target.value)} />
      <input type="username" value={userName} onChange={(e) => setUserName(e.target.value)} />
      <input type="email" value={email} onChange={(e) => setEmail(e.target.value)} />
      <input type="password" value={password} onChange={(e) => setPassword(e.target.value)} />
      <button type="submit">Submit</button>
    </form>
  );
};

export default UserRegister;
```

---
layout: intro
---

# Vamos melhorar isso?

---
transition: slide-up
---

```tsx{3-8|10|10,16-18|10,16-28}{maxHeight:'80vh'}
import { useState } from "react";

interface User {
 userName: string;
 name: string;
 password: string;
 email: string;
}
export const UserRegister = () => {
 const [inputs, setInputs] = useState({} as User);

 const handleSubmit: React.FormEventHandler<HTMLFormElement> = (e) => {
  e.preventDefault();
 };

 const handleSetInput = (inputName: keyof User) => (e: React.ChangeEvent<HTMLInputElement>) => {
  setInputs({ ...inputs, [inputName]: e.target.value });
 };

 return (
  <form onSubmit={handleSubmit}>
   <input type="name" value={inputs["name"]} onChange={handleSetInput("name")} />
   <input type="username" value={inputs["userName"]} onChange={handleSetInput("userName")} />
   <input type="email" value={inputs["email"]} onChange={handleSetInput("email")} />
   <input type="password" value={inputs["password"]} onChange={handleSetInput("password")} />
   <button type="submit">Submit</button>
  </form>
 );
};

export default UserRegister;

```

---

# Legal, mas ainda estamos tratando isso de forma literal!
Estamos construindo os formulários em vez de considerar apenas os dados

Para resolver isso podemos separar isso em algumas partes, primeiro separamos nossa entidade *User*

```tsx {3-8|10-30|43-51}{lines:true,startLine:1,maxHeight:'80vh'}
import { useState } from "react";

interface User {
 userName: string;
 name: string;
 password: string;
 email: string;
}

const UserScheme: {
  name: keyof User;
  type: 'text' | 'email' | 'password'
}[] = [
  {
   name: "name",
   type: 'text'
  },
  {
   name: "userName",
   type: 'text'
  },
  {
   name: "email",
   type: 'email'
  },
  {
   name: "password",
   type: 'password'
  }
]

export const UserRegister = () => {
 const [inputs, setInputs] = useState({} as User);

 const handleSubmit: React.FormEventHandler<HTMLFormElement> = (e) => {
  e.preventDefault();
 };

 const handleSetInput = (inputName: keyof User) => (e: React.ChangeEvent<HTMLInputElement>) => {
  setInputs({ ...inputs, [inputName]: e.target.value });
 };

 return (
  <form onSubmit={handleSubmit}>
   {UserScheme.map((input) => (
    <input key={input.name} type={input.type} value={inputs[input.name]} onChange={handleSetInput(input.name)} />
   ))}
   <button type="submit">Submit</button>
  </form>
 );
};

export default UserRegister;
```

---

# Isolando atividades

````md magic-move 
```tsx
export const UserRegister = () => {
 const [inputs, setInputs] = useState({} as User);

 const handleSubmit: React.FormEventHandler<HTMLFormElement> = (e) => {
  e.preventDefault();
 };

 const handleSetInput = (inputName: keyof User) => (e: React.ChangeEvent<HTMLInputElement>) => {
  setInputs({ ...inputs, [inputName]: e.target.value });
 };

 return (
  <form onSubmit={handleSubmit}>
   {UserScheme.map((input) => (
    <input key={input.name} type={input.type} value={inputs[input.name]} onChange={handleSetInput(input.name)} />
   ))}
   <button type="submit">Submit</button>
  </form>
 );
};

export default UserRegister;
```

```tsx
const useForm = () => {
 const [inputs, setInputs] = useState({} as User);
 const handleSetInput = (inputName: keyof User) => (e: React.ChangeEvent<HTMLInputElement>) => {
  setInputs({ ...inputs, [inputName]: e.target.value });
 };

 return { inputs, handleSetInput };
};

export const UserRegister = () => {
 const { inputs, handleSetInput } = useForm();
 const handleSubmit: React.FormEventHandler<HTMLFormElement> = (e) => {
  e.preventDefault();
 };

 return (
  <form onSubmit={handleSubmit}>
   {UserScheme.map((input) => (
    <input key={input.name} type={input.type} value={inputs[input.name]} onChange={handleSetInput(input.name)} />
   ))}
   <button type="submit">Submit</button>
  </form>
 );
};

export default UserRegister;
```
```tsx
const useForm = () => {
 const [values, setValues] = useState({} as User);
 const onChange = (inputName: keyof User) => (e: React.ChangeEvent<HTMLInputElement>) => {
  setValues({ ...values, [inputName]: e.target.value });
 };

 return { values, onChange };
};

export const UserRegister = () => {
 const { values, onChange } = useForm();
 const handleSubmit: React.FormEventHandler<HTMLFormElement> = (e) => {
  e.preventDefault();
 };

 return (
  <form onSubmit={handleSubmit}>
   {UserScheme.map((input) => (
    <input key={input.name} type={input.type} value={values[input.name]} onChange={onChange(input.name)} />
   ))}
   <button type="submit">Submit</button>
  </form>
 );
};

export default UserRegister;
```
```tsx {1-9|11-25}{lines:'true',startLine:1,maxHeight:'80vh'}
const useForm = () => {
 const [values, setValues] = useState({} as User);
 const onChange = (inputName: keyof User) => (e: React.ChangeEvent<HTMLInputElement>) => {
  setValues({ ...values, [inputName]: e.target.value });
 };

 return { values, onChange };
};

const Form = ({
 scheme,
 handleSubmit,
}: {
 scheme: {
  name: keyof User;
  type: "text" | "email" | "password";
 }[];
 handleSubmit: React.FormEventHandler<HTMLFormElement>;
}) => {
 const { values, onChange } = useForm();
 return (
  <form onSubmit={handleSubmit}>
   {scheme.map((input) => (
    <input key={input.name} type={input.type} value={values[input.name]} onChange={onChange(input.name)} />
   ))}
   <button type="submit">Submit</button>
  </form>
 );
};

export const UserRegister = () => {
 const handleSubmit: React.FormEventHandler<HTMLFormElement> = (e) => {
  e.preventDefault();
 };

 return <Form scheme={UserScheme} handleSubmit={handleSubmit} />;
};
```
````

---

```tsx {1-9|10-29|31-37}{lines:true,startLine:1,maxHeight:'80vh'}
const useForm = () => {
 const [values, setValues] = useState({} as User);
 const onChange = (inputName: keyof User) => (e: React.ChangeEvent<HTMLInputElement>) => {
  setValues({ ...values, [inputName]: e.target.value });
 };

 return { values, onChange };
};

const Form = ({
 scheme,
 handleSubmit,
}: {
 scheme: {
  name: keyof User;
  type: "text" | "email" | "password";
 }[];
 handleSubmit: React.FormEventHandler<HTMLFormElement>;
}) => {
 const { values, onChange } = useForm();
 return (
  <form onSubmit={handleSubmit}>
   {scheme.map((input) => (
    <input key={input.name} type={input.type} value={values[input.name]} onChange={onChange(input.name)} />
   ))}
   <button type="submit">Submit</button>
  </form>
 );
};

export const UserRegister = () => {
 const handleSubmit: React.FormEventHandler<HTMLFormElement> = (e) => {
  e.preventDefault();
 };

 return <Form scheme={UserScheme} handleSubmit={handleSubmit} />;
};
```

---

# Criamos componentes quase reutilizáveis
Vamos os deixar reutilizáveis de fato

````md magic-move

```tsx
const useForm = () => {
 const [values, setValues] = useState({} as User);
 const onChange = (inputName: keyof User) => (e: React.ChangeEvent<HTMLInputElement>) => {
  setValues({ ...values, [inputName]: e.target.value });
 };

 return { values, onChange };
};
```

```tsx
const useForm = <T extends Record<string, unknown>>() => {
 const [values, setValues] = useState({} as T);
 const onChange = (inputName: keyof T) => (e: React.ChangeEvent<HTMLInputElement>) => {
  setValues({ ...values, [inputName]: e.target.value });
 };

 return { values, onChange };
};
```

```tsx
...
```

````
---

# Criamos componentes quase reutilizáveis

Vamos os deixar reutilizáveis de fato

````md magic-move

```tsx
const Form = ({
 scheme,
 handleSubmit,
}: {
 scheme: {
  name: keyof User;
  type: "text" | "email" | "password";
 }[];
 handleSubmit: React.FormEventHandler<HTMLFormElement>;
}) => {
 const { values, onChange } = useForm();
 return (
  <form onSubmit={handleSubmit}>
   {scheme.map((input) => (
    <input key={input.name} type={input.type} value={values[input.name]} onChange={onChange(input.name)} />
   ))}
   <button type="submit">Submit</button>
  </form>
 );
};
```

```tsx
const Form = <T extends Record<string, unknown>>({
 scheme,
 handleSubmit,
}: {
 scheme: {
  name: keyof T;
  type: "text" | "email" | "password";
 }[];
 handleSubmit: React.FormEventHandler<HTMLFormElement>;
}) => {
 const { values, onChange } = useForm<T>();
 return (
  <form onSubmit={handleSubmit}>
   {scheme.map((input) => (
    <input 
      key={input.name as string}
      type={input.type}
      value={values[input.name] as string}
      onChange={onChange(input.name)} />
   ))}
   <button type="submit">Submit</button>
  </form>
 );
};
```

````