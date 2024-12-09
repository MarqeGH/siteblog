---
title: 'Setting up twitter(x) login with NextAuth + neon database storage with Vercel'
date: 2024-11-18T10:58:08+10:00
draft: false
---

# Setting up twitter(x) login with NextAuth + Neon database storage with Vercel + a small message widget

## Setting up NextAuth

To implement NextAuth on your website, first install Next-Auth
```bash
npm install next-auth
```

**--- Error message START, move to error message troubleshooting if encountered ---**
If you're having problems here, it might be because your ICU4C library is outdated.
[What is ICU](/what-is-icu) (bonus info)

To fix this, reinstall ICU
```bash
brew reinstall icu4c
```

Link it
```bash
brew link icu4c --force
```
**--- Error message END ---**

Once you have installed Next-Auth, create a .env.local file in your project's root.

In project root, set up a secret token for NextAuth
```bash
npx auth secret
```
This will put a NEXT_SECRET variable in your .env.local file.
Add a NEXT_URL="http://localhost:3000" variable.
- In vercel put in your site where you will implement authentication.

In your project, create the folders
- app/api/auth/[...nextauth]
	- [...nextauth] is a catch-all dynamic route that handles any sub-routes under auth that processes requests for nextauth.js.
		- Signing in users
		- Signing out users
		- Handling OAuth callbacks
		- Refreshing tokens
		- Accessing user session information

## Setting up Twitter authentication

- Create a Twitter Developer Account
- Create a project, making sure to set the following settings:
	- App permissions:
		- Read and write
		- Don't request email from users
	- Type of App:
		- Web App, Automated App or Bot
	- App info:
		- Callback url
			- http://localhost:3000/api/auth/callback/twitter
			- any live site you want it on, ex. https://yoursite.com/api/auth/callback/twitter
		- Website url
			- Some site to identify you
		- ToS
			- If you want / if you collect user data
		- Privacy policy
			- If you want / If you collect user data
Congrats, you now have API keys! 
	Take note of the Client ID and Secret. You will need these.

### Set up a Neon Database with Vercel
On your Vercel project, go to storage and set up a free Neon database. Access the Neon console linked in your vercel project and go to the SQL Editor.

Create a table in your neon database by typing the below and running the code:
```sql
CREATE TABLE comments (
  id SERIAL PRIMARY KEY,
  content TEXT NOT NULL,
  username TEXT NOT NULL,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);
```
This will create a table called comments that will store the comment id, the content of any messages written included, the username and the time it was created.

### Set up Twitter Authentication Types

Define the Twitter profile structure
Create a file called types.ts – this file will contain undefined types to satisfy typescript
```ts
export interface TwitterProfile {
    data: {
      id: string;
      name: string;
      username: string;
      // add any more here
    };
}
```

### Configure NextAuth with Twitter Provider

Create the authentification configuration

in app/api/[...nextauth]/route.ts:

```ts
import NextAuth, { type NextAuthOptions } from "next-auth";
import TwitterProvider from "next-auth/providers/twitter";

import { TwitterProfile } from "@/app/types/types";
// replace with wherever you put your types.ts file

// 1. Extend NextAuth session to include Twitter username (and other)
declare module "next-auth" {
    interface Session {
        user: {
            username?: string;
            name?: string | null;
            email?: string | null;
            image?: string | null;
            // any other
        };
    }
}

// 2. Set up Twitter Provider
const authOptions: NextAuthOptions = {
    providers: [
        TwitterProvider({
            clientId: process.env.TWITTER_CLIENT_ID as string,
            clientSecret: process.env.TWITTER_CLIENT_SECRET as string,
            version: "2.0",
            // make sure you have configured your twitter dev project to be 2.0
            authorization: {
                params: {
                    scope: "users.read tweet.read offline.access",
                }
            }
        }),
    ],
    callbacks: {
        // 3. Capture username during JWT creation
        async jwt({ token, profile }) {
            if (profile) {
                const twitterProfile = profile as TwitterProfile;
                token.username = twitterProfile.data.username;
            }
            return token;
        },
        // 4. Add username to session
        async session({ session, token }) {
            if (session.user) {
                session.user.username = token.username as string;
            }
            return session;
        }
    }
};
```


### Create database operations

Create an actions.ts file to do database operations. I have added some optional comment features but you can do any other database actions using similar functions.
[Database operations](/database-operations) (bonus info)(*coming soon*)

```ts
'use server';

export async function createComment(comment: string, twitterHandle: string) {
  try {
    const sql = neon(process.env.DATABASE_URL!);
    
    const result = await sql`
      INSERT INTO comments (content, username, created_at) 
      VALUES (${comment}, ${twitterHandle}, NOW())
      RETURNING *
    `;
    
    return { 
      success: true, 
      data: {
        text: result[0].content,
        username: result[0].username
      }
    };
  } catch (error) {
    return { success: false, error: 'Failed to create comment' };
  }
}

export async function getMessages() {
  const sql = neon(process.env.DATABASE_URL!);
  const messages = await sql`
    SELECT content, username FROM comments
    ORDER BY created_at DESC
  `;
  return messages.map(msg => ({
    text: msg.content,
    username: msg.username
  }));
}
```

### Optional: Create a message list component for your users to write messages

Create a MessageList.tsx file in /components/
```tsx
export default function MessageList({ messages }: MessageListProps) {
  return (
    <div className="grid grid-cols-3 gap-4">
      {[...messages].map((message, index) => (
        <div key={index} className="p-4 border rounded-lg shadow-sm">
          <p className="break-words mb-2">{message.text}</p>
          <a href={`https://x.com/${message.username}`}>
            @{message.username}
          </a>
        </div>
      ))}
    </div>
  );
}
```

### Put it all together on the main page
```tsx
export default function Home() {
  const { data: session, status } = useSession();
  const [messages, setMessages] = useState<{ text: string, username: string }[]>([]);

  // Load messages on component mount
  useEffect(() => {
    async function fetchMessages() {
      const fetchedMessages = await getMessages();
      setMessages(fetchedMessages);
    }
    fetchMessages();
  }, []);

  // Handle message creation
  async function create(formData: FormData) {
    const comment = formData.get('comment') as string;
    if (!comment?.trim()) return;

    const result = await createComment(comment, session?.user?.username || 'anonymous');
    if (result.success) {
      setMessages(prev => [result.data, ...prev]);
    }
  }

  return (
    <div>
      {session ? (
        <>
          <h2>Welcome, {session.user.username}!</h2>
          <form onSubmit={(e) => {
            e.preventDefault();
            create(new FormData(e.currentTarget));
          }}>
            <input type="text" name="comment" placeholder="Write a message" />
            <button type="submit">Submit</button>
          </form>
        </>
      ) : (
        <button onClick={() => signIn('twitter')}>
          Sign in with Twitter
        </button>
      )}
      <MessageList messages={messages} />
    </div>
  );
}
```

### Make sure your .env.local file is complete

```env
TWITTER_CLIENT_ID=your_twitter_client_id
TWITTER_CLIENT_SECRET=your_twitter_client_secret
DATABASE_URL=your_neon_database_url
NEXTAUTH_SECRET=your_nextauth_secret
NEXTAUTH_URL=http://localhost:3000
```
remember that this file only stores the environment variables locally, and you should put the variables in your hosted environment when you launch. 


Now you should be able to test locally and launch globally (on the world wide www's) your twitter authentication, with some database fun.