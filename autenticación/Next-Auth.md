# Autenticación en Next

## Next-Auth

Ver documentación de [Next-Auth](https://nextjs.org/learn/dashboard-app/adding-authentication)

1. Instalar las ***Next-Auth*** en el proyecto con el siguiente comando, este ya es compatible con ***Next.js 14*** o versiones superiores

```bash
npm install next-auth@beta
```

2. Generar una *Secret Key* para la aplicación. Esta *key* es utilizada para encriptar las *cookies* que aseguran una sesión segura para el usuario. Puedes hacer esto ejecutando el siguiente comando

```bash
npx auth secret
```

3. Luego, dentro del archivo `.env`, generar la variable `AUTH_SECRET` y agregar el valor generado

```env
AUTH_SECRET=la clave generada
```
Si despliegas el proyecto en alguna plataforma, verificar como actualizar las variables de entorno

4. En la carpeta raiz del proyecto, generar un archivo que tendrá la configuración de la autentificación `src/auth.js`

```typescript
import NextAuth from "next-auth";
 
export const { handlers, signIn, signOut, auth } = NextAuth({
  providers: [],
})
```

5. Crea el archivo del manejador de rutas en `src/app/api/auth/[...nextauth]/route.ts`

```typescript
import { handlers } from "@/auth" // Referring to the auth.ts we just created
export const { GET, POST } = handlers
```

6. Creamos una página en la aplicación para la ***autentificación*** del usuario `src/app/auth/login/page.tsx`, junto a esta un archivo con su formulario `src/app/auth/login/ui/LoginForm.tsx`, este ultimo será un componente del lado del cliente

***page.tsx***

```typescript

import { titleFont } from '@/config/fonts';
import { LoginForm } from './ui/LoginForm';

export default function LoginPage() {
  return (
    <div className="flex flex-col min-h-screen pt-32 sm:pt-52">

      <h1 className={ `${ titleFont.className } text-4xl mb-5` }>Ingresar</h1>

      <LoginForm />
    </div>
  );
}
```

***LoginForm.tsx***
```typescript
"use client";

import { useEffect } from 'react';
import Link from "next/link";
import { useFormState, useFormStatus } from "react-dom";

import { authenticate } from "@/actions";
import { IoInformationOutline } from "react-icons/io5";
import clsx from 'clsx';
// import { useRouter } from 'next/navigation';

export const LoginForm = () => {


  // const router = useRouter();
  const [state, dispatch] = useFormState(authenticate, undefined);
  
  console.log(state);

  useEffect(() => {
    if ( state === 'Success' ) {
      // redireccionar
      // router.replace('/');
      window.location.replace('/');
    }

  },[state]);



  return (
    <form action={dispatch} className="flex flex-col">
      <label htmlFor="email">Correo electrónico</label>
      <input
        className="px-5 py-2 border bg-gray-200 rounded mb-5"
        type="email"
        name="email"
      />

      <label htmlFor="email">Contraseña</label>
      <input
        className="px-5 py-2 border bg-gray-200 rounded mb-5"
        type="password"
        name="password"
      />

      <div
        className="flex h-8 items-end space-x-1"
        aria-live="polite"
        aria-atomic="true"
      >
        {state === "CredentialsSignin" && (
          <div className="flex flex-row mb-2">
            <IoInformationOutline className="h-5 w-5 text-red-500" />
            <p className="text-sm text-red-500">
              Credenciales no son correctas
            </p>
          </div>
        )}
      </div>

        <LoginButton />
      {/* <button type="submit" className="btn-primary">
        Ingresar
      </button> */}

      {/* divisor l ine */}
      <div className="flex items-center my-5">
        <div className="flex-1 border-t border-gray-500"></div>
        <div className="px-2 text-gray-800">O</div>
        <div className="flex-1 border-t border-gray-500"></div>
      </div>

      <Link href="/auth/new-account" className="btn-secondary text-center">
        Crear una nueva cuenta
      </Link>
    </form>
  );
};

function LoginButton() {
  const { pending } = useFormStatus();

  return (
    <button 
      type="submit" 
      className={ clsx({
        "btn-primary": !pending,
        "btn-disabled": pending
      })}
      disabled={ pending }
      >
      Ingresar
    </button>
  );
}
```

7. Creamos una página en la aplicación para la ***creación de nuevo usuario*** del usuario `src/app/auth/new-account/page.tsx`, junto a esta un archivo con su formulario `src/app/auth/new-account/ui/RegisterForm.tsx`, este ultimo será un componente del lado del cliente

***page.tsx***

```typescript
import { titleFont } from '@/config/fonts';
import { RegisterForm } from './ui/RegisterForm';

export default function NewAccountPage() {
  return (
    <div className="flex flex-col min-h-screen pt-32 sm:pt-52">

      <h1 className={ `${ titleFont.className } text-4xl mb-5` }>Nueva cuenta</h1>

      <RegisterForm />
    </div>
  );
}
```

***RegisterForm.tsx***

```typescript
"use client";

import clsx from 'clsx';
import Link from 'next/link';
import { SubmitHandler, useForm } from 'react-hook-form';

import { login, registerUser } from '@/actions';
import { useState } from 'react';


type FormInputs = {
  name: string;
  email: string;
  password: string;  
}



export const RegisterForm = () => {

  const [errorMessage, setErrorMessage] = useState('')
  const { register, handleSubmit, formState: {errors} } = useForm<FormInputs>();

  const onSubmit: SubmitHandler<FormInputs> = async(data) => {
    setErrorMessage('');
    const { name, email, password } = data;
    
    // Server action
    const resp = await registerUser( name, email, password );

    if ( !resp.ok ) {
      setErrorMessage( resp.message );
      return;
    }

    await login( email.toLowerCase(), password );
    window.location.replace('/');


  }


  return (
    <form onSubmit={ handleSubmit( onSubmit ) }  className="flex flex-col">

      {/* {
        errors.name?.type === 'required' && (
          <span className="text-red-500">* El nombre es obligatorio</span>
        )
      } */}


      <label htmlFor="email">Nombre completo</label>
      <input
        className={
          clsx(
            "px-5 py-2 border bg-gray-200 rounded mb-5",
            {
              'border-red-500': errors.name
            }
          )
        }
        type="text"
        autoFocus
        { ...register('name', { required: true }) }
      />

      <label htmlFor="email">Correo electrónico</label>
      <input
        className={
          clsx(
            "px-5 py-2 border bg-gray-200 rounded mb-5",
            {
              'border-red-500': errors.email
            }
          )
        }
        type="email"
        { ...register('email', { required: true, pattern: /^\S+@\S+$/i }) }
      />

      <label htmlFor="email">Contraseña</label>
      <input
        className={
          clsx(
            "px-5 py-2 border bg-gray-200 rounded mb-5",
            {
              'border-red-500': errors.password
            }
          )
        }
        type="password"
        { ...register('password', { required: true, minLength: 6 }) }
      />

      
        <span className="text-red-500">{ errorMessage } </span>
        
      

      <button className="btn-primary">Crear cuenta</button>

      {/* divisor l ine */}
      <div className="flex items-center my-5">
        <div className="flex-1 border-t border-gray-500"></div>
        <div className="px-2 text-gray-800">O</div>
        <div className="flex-1 border-t border-gray-500"></div>
      </div>

      <Link href="/auth/login" className="btn-secondary text-center">
        Ingresar
      </Link>
    </form>
  );
};
```

8. Hay que hacer referencia de estas pantallas personalizadas en el archivo `src/auth.ts`

```typescript
import NextAuth from "next-auth";
import Credentials from "next-auth/providers/credentials";
import { z } from "zod";
 
export const { handlers, signIn, signOut, auth } = NextAuth({

    pages: {
        signIn: '/auth/login',              // Pagina de inicio de sesión
        newUser: '/auth/new-account',       // Página para creación de nuevo usuario
    },

    callbacks: {

    }

    providers: [

    ],
})
```

9. Para agregar el proveedor de credenciales de la aplicación, para que el usuario pueda autentificarse con un *e-mail* y *contraseña*, primero deberemos instalar el siguiente paquete que nos sirve para validar la info del formulario que sea correcta

```bash
npm i zod
```

10. Luego, en el archivo `src/auth.js`, integraremos en el área de los ***providers*** el proveedor de credenciales

```typescript
import NextAuth from "next-auth";
import Credentials from "next-auth/providers/credentials";
import { z } from "zod";
 
export const { handlers, signIn, signOut, auth } = NextAuth({
  providers: [
    Credentials({
        async authorize(credentials) {
            const parsedCredentials = z
                .object({ email: z.string().email(), password: z.string().min(6) })
                .safeParse(credentials);

            if ( !parsedCredentials.success ) return null;

            const { email, password } = parsedCredentials.data;

            // Buscar el correo

            // Comparar las contraseñas

            // Regresar el usuario

        }
    })
  ],
})
```

11. Ahora deberemos crear en la carpeta raiz del proyecto, la carpeta `src/actions/auth` para dentro de esta crear los siguientes arvhivos.

***login.ts***
```typescript
'use server';


import { signIn } from '@/auth';
import { sleep } from '@/utils';
 
// ...
 
export async function authenticate(
  prevState: string | undefined,
  formData: FormData,
) {
  try {

    // await sleep(2);
    
    await signIn('credentials', {
      ...Object.fromEntries(formData),
      redirect: false,
    });

    return 'Success';


  } catch (error) {
    console.log(error);

    return 'CredentialsSignin'


  }
}


export const login = async(email:string, password: string) => {

  try {

    await signIn('credentials',{ email, password })

    return {ok: true};
    
  } catch (error) {
    console.log(error);
    return {
      ok: false,
      message: 'No se pudo iniciar sesión'
    }
    
  }


}
```

***logout.ts***
```typescript
'use server';

import { signOut } from '@/auth';


export const logout = async() => {

  await signOut();


}
```

***register.ts***
```typescript
'use server';

import prisma from '@/lib/prisma';
import bcryptjs from 'bcryptjs'


export const registerUser = async( name: string, email: string, password: string ) => {


  try {
    
    const user = await prisma.user.create({
      data: {
        name: name,
        email: email.toLowerCase(),
        password: bcryptjs.hashSync( password ),
      },
      select: {
        id: true,
        name: true,
        email: true,
      }
    })

    return {
      ok: true,
      user: user,
      message: 'Usuario creado'
    }

  } catch (error) {
    console.log(error);

    return {
      ok: false,
      message: 'No se pudo crear el usuario'
    }
  }

  

}
```

12. Ahora crearemos un modelo de Usuarios en Prisma, este es un ejemplo

```prisma
enum Role {
  admin
  user
}

model User {
  id            String    @id @default(uuid())
  name          String
  email         String    @unique
  emailVerified DateTime?
  password      String
  role          Role      @default(user)
  image         String?
}
```