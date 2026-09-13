<h1 align="center">💪 AI Fitness Assistant 🤖</h1>

<p align="center">By Syakeer</p>

![Demo App](/public/screenshot-for-readme.png)

CodeFlex AI is a web app that makes a personal workout plan and a personal diet plan. The user speaks with an AI voice coach. The coach asks about goals, body, injuries, and food limits. Then the app writes the plan and shows it on the profile page.

## Features

- **Voice AI coach**: The user speaks with an AI coach in the browser. The coach asks questions one at a time.
- **Personal workout plan**: The plan uses the fitness level, the injuries, the free days, and the goal.
- **Personal diet plan**: The plan uses the daily calorie target and the dietary restrictions.
- **Sign in**: The user signs in with GitHub, Google, or email and password.
- **Plan history**: The user can keep many plans. Only the newest plan is active.
- **Responsive design**: The pages work on phones, tablets, and computers.

## Tech stack

| Part | Tool | What it does in this app |
|---|---|---|
| Frontend | Next.js 15, React 19 | Shows the pages |
| Styles | Tailwind CSS 4, shadcn/ui | Gives the look of the pages |
| Sign in | Clerk | Keeps user accounts |
| Voice | Vapi | Runs the voice call with the AI coach |
| AI | Google Gemini | Writes the workout plan and the diet plan |
| Database and backend | Convex | Keeps users and plans, and runs the backend code |

## How the app works

1. The user signs up. Clerk sends a webhook to Convex. Convex adds the user to the `users` table.
2. The user opens `/generate-program` and starts a call with the Vapi assistant.
3. The assistant asks 8 questions. Then it sends the answers to the Convex endpoint `/vapi/generate-program`.
4. Convex asks Gemini for a workout plan and a diet plan.
5. Convex saves the new plan in the `plans` table. It marks all older plans of that user as not active.
6. The call ends. The app opens `/profile` and shows the plan.

---

# Build it step by step

Do the steps in this order. Some steps need a value from an earlier step.

Time: about 45 to 60 minutes the first time.

## Step 0: Get the tools and accounts

Install these tools on your computer:

| Tool | Version | Check command |
|---|---|---|
| [Node.js](https://nodejs.org) | 20 or newer | `node -v` |
| [Git](https://git-scm.com) | any | `git --version` |

Make a free account on each of these services:

| Service | Sign-up page |
|---|---|
| Clerk | https://dashboard.clerk.com/sign-up |
| Convex | https://dashboard.convex.dev |
| Vapi | https://dashboard.vapi.ai |
| Google AI Studio (for Gemini) | https://aistudio.google.com |

Keep a text file open during the setup. You will copy about 10 values into it.

## Step 1: Get the code

1. Open a terminal.
2. Clone the repository:

   ```shell
   git clone https://github.com/SyakeerRahman/workout-ai.git
   ```

3. Go into the project folder:

   ```shell
   cd workout-ai
   ```

4. Install the packages:

   ```shell
   npm install
   ```

## Step 2: Set up Clerk (sign in)

1. Go to https://dashboard.clerk.com.
2. Click **Create application**.
3. Type a name, for example `CodeFlex AI`.
4. Turn on the sign-in options: **Email**, **Google**, and **GitHub**.
5. Click **Create application**.
6. Open the **API keys** page. Copy these 2 values to your text file:

   | Clerk value | Save it as |
   |---|---|
   | Publishable key (starts with `pk_`) | `NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY` |
   | Secret key (starts with `sk_`) | `CLERK_SECRET_KEY` |

7. Open https://dashboard.clerk.com/apps/setup/convex.
8. Select your application, then click **Activate Convex integration**.
9. Copy the **Frontend API URL**. It looks like `https://verb-noun-00.clerk.accounts.dev`. Save it as `CLERK_JWT_ISSUER_DOMAIN`.

## Step 3: Get a Gemini API key

1. Go to https://aistudio.google.com/apikey.
2. Click **Create API key**.
3. Copy the key. Save it as `GEMINI_API_KEY`.

## Step 4: Set up Convex (database and backend)

1. In the project folder, run:

   ```shell
   npx convex dev
   ```

2. Log in to Convex when the terminal asks.
3. Choose **create a new project**. Type a name, for example `workout-ai`.
4. Convex creates the file `.env.local` in the project folder. The file contains `CONVEX_DEPLOYMENT` and `NEXT_PUBLIC_CONVEX_URL`.
5. The terminal can show an error about `CLERK_JWT_ISSUER_DOMAIN`. This error is expected. Press `Ctrl+C` to stop Convex.
6. Set the 2 backend secrets. Use your own values:

   ```shell
   npx convex env set CLERK_JWT_ISSUER_DOMAIN https://verb-noun-00.clerk.accounts.dev
   npx convex env set GEMINI_API_KEY your-gemini-key
   ```

7. Start Convex again:

   ```shell
   npx convex dev
   ```

8. Wait for the message `Convex functions ready!`. Keep this terminal open.
9. Find your **HTTP Actions URL**. Open `.env.local` and copy the `NEXT_PUBLIC_CONVEX_URL` value. Change `.convex.cloud` to `.convex.site`.

   Example: `https://happy-otter-123.convex.cloud` becomes `https://happy-otter-123.convex.site`.

   You can also find this URL in the Convex dashboard under **Settings** > **URL & Deploy Key**.

## Step 5: Connect Clerk to Convex with a webhook

This webhook copies each new Clerk user into the Convex `users` table.

1. In the Clerk dashboard, open **Configure** > **Webhooks**.
2. Click **Add Endpoint**.
3. In **Endpoint URL**, type your HTTP Actions URL and `/clerk-webhook`:

   ```text
   https://happy-otter-123.convex.site/clerk-webhook
   ```

4. Under **Subscribe to events**, select `user.created` and `user.updated`.
5. Click **Create**.
6. On the endpoint page, copy the **Signing Secret** (starts with `whsec_`).
7. Open a second terminal in the project folder. Set the secret in Convex:

   ```shell
   npx convex env set CLERK_WEBHOOK_SECRET whsec_your-secret
   ```

## Step 6: Set up the Vapi voice assistant

> **Note:** Vapi stopped Workflows on 19 August 2026. This app now uses a Vapi **Assistant** with an **API Request** tool.

### 6a. Get the Vapi public key

1. Go to https://dashboard.vapi.ai.
2. Open the **API Keys** page.
3. Copy the **Public Key**. Save it as `NEXT_PUBLIC_VAPI_API_KEY`.

Do not use the private key. The browser can see this value.

### 6b. Make the tool that sends the answers to Convex

1. In the Vapi dashboard, open **Tools**.
2. Click **Create Tool**. Select **API Request**.
3. Set the tool name to `generate_program`.
4. Set the description to `Creates the workout plan and the diet plan after all questions are answered.`
5. Under **Base Configuration**, set these values:

   | Field | Value |
   |---|---|
   | Request URL | `https://happy-otter-123.convex.site/vapi/generate-program` (use your own URL) |
   | Request HTTP Method | `POST` |

6. Under **Request Body**, add these 8 properties. Set the type of each property to **String**, and make each property required:

   | Property | Description |
   |---|---|
   | `age` | Age of the user in years |
   | `height` | Height of the user, with the unit |
   | `weight` | Weight of the user, with the unit |
   | `injuries` | Injuries or body limits, or "none" |
   | `workout_days` | Days per week that the user can work out |
   | `fitness_goal` | Main goal, for example "lose weight" or "build muscle" |
   | `fitness_level` | "beginner", "intermediate", or "advanced" |
   | `dietary_restrictions` | Allergies or food limits, or "none" |

7. Turn on **Lock schema (no additional properties)**.
8. Under **Static Body Fields**, click **Add Field**. Add this field:

   | Key | Type | Value |
   |---|---|---|
   | `user_id` | String | `{{user_id}}` |

   The app sends `user_id` when the call starts. Vapi puts the value in this field.

9. If the tool has a timeout setting, set it to `60` seconds. Gemini needs time to write 2 plans.
10. Click **Publish**.

### 6c. Make the assistant

1. In the Vapi dashboard, open **Assistants**.
2. Click **Create Assistant**. Select the blank template.
3. Set the name to `CodeFlex AI`.
4. Set the **First Message**:

   ```text
   Hi {{full_name}}, I am your AI fitness coach. I will ask you a few short questions to build your plan. Are you ready?
   ```

5. Set the **System Prompt**:

   ```text
   You are CodeFlex AI, a friendly fitness and nutrition coach.
   Ask the user these questions, one at a time:
   1. How old are you?
   2. How tall are you?
   3. How much do you weigh?
   4. Do you have any injuries or body limits?
   5. How many days per week can you work out?
   6. What is your main fitness goal?
   7. What is your fitness level: beginner, intermediate, or advanced?
   8. Do you have any allergies or dietary restrictions?
   Keep each reply short. Do not give advice during the questions.
   When you have all 8 answers, call the generate_program tool.
   Then tell the user that the plan is ready on the profile page, say goodbye, and end the call.
   ```

6. Open the **Tools** section of the assistant. Add `generate_program`.
7. Add the built-in **End Call** tool. The app opens the profile page when the call ends.
8. Click **Publish**.
9. Copy the **Assistant ID** from the top of the assistant page. Save it as `NEXT_PUBLIC_VAPI_ASSISTANT_ID`.

## Step 7: Fill in the .env.local file

Open `.env.local` in the project folder. Convex already added 2 lines. Add the other lines. The file must look like this, with your own values:

```shell
# Convex (added by "npx convex dev")
CONVEX_DEPLOYMENT=dev:happy-otter-123
NEXT_PUBLIC_CONVEX_URL=https://happy-otter-123.convex.cloud

# Clerk
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_test_...
CLERK_SECRET_KEY=sk_test_...
NEXT_PUBLIC_CLERK_SIGN_IN_URL=/sign-in
NEXT_PUBLIC_CLERK_SIGN_UP_URL=/sign-up

# Vapi
NEXT_PUBLIC_VAPI_API_KEY=your-vapi-public-key
NEXT_PUBLIC_VAPI_ASSISTANT_ID=your-assistant-id
```

Git ignores `.env.local`. Do not commit this file.

The other 3 secrets are on the Convex server, not in this file. To check them, run `npx convex env list`:

| Convex environment variable | From |
|---|---|
| `CLERK_JWT_ISSUER_DOMAIN` | Step 2 |
| `GEMINI_API_KEY` | Step 3 |
| `CLERK_WEBHOOK_SECRET` | Step 5 |

## Step 8: Run the app and test it

1. Make sure that `npx convex dev` runs in the first terminal.
2. In the second terminal, start the website:

   ```shell
   npm run dev
   ```

3. Open http://localhost:3000.
4. Click **Sign up** and make an account.
5. In the Convex dashboard, open **Data** > **users**. Make sure that your user is in the table. If it is not there, go to "Problems and fixes" below.
6. Open http://localhost:3000/generate-program.
7. Click **Start Call**. Allow the browser to use the microphone.
8. Answer the 8 questions.
9. Wait for the call to end. The app opens the profile page and shows your new plan.
10. In the Convex dashboard, open **Data** > **plans**. Make sure that the new plan is in the table.

## Step 9: Put the app online with Vercel (optional)

1. Push the code to your GitHub account.
2. Go to https://vercel.com/new and import the repository.
3. In the Convex dashboard, open your project. Select the **Production** deployment.
4. Open **Settings** > **URL & Deploy Key**. Click **Generate Production Deploy Key**. Copy the key.
5. In Vercel, set **Build Command** to:

   ```shell
   npx convex deploy --cmd 'npm run build'
   ```

6. In Vercel, add these environment variables:

   | Name | Value |
   |---|---|
   | `CONVEX_DEPLOY_KEY` | The key from item 4 |
   | `NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY` | From Clerk |
   | `CLERK_SECRET_KEY` | From Clerk |
   | `NEXT_PUBLIC_CLERK_SIGN_IN_URL` | `/sign-in` |
   | `NEXT_PUBLIC_CLERK_SIGN_UP_URL` | `/sign-up` |
   | `NEXT_PUBLIC_VAPI_API_KEY` | From Vapi |
   | `NEXT_PUBLIC_VAPI_ASSISTANT_ID` | From Vapi |

   The build command sets `NEXT_PUBLIC_CONVEX_URL`. You do not add it.

7. In the Convex dashboard, set `CLERK_JWT_ISSUER_DOMAIN`, `GEMINI_API_KEY`, and `CLERK_WEBHOOK_SECRET` on the **Production** deployment too.
8. Click **Deploy** in Vercel.
9. Change the URLs to the production URL:
   - The Clerk webhook endpoint (Step 5).
   - The `generate_program` tool Request URL in Vapi (Step 6b).

To build and start the app on your own computer without Vercel, run:

```shell
npm run build
npm run start
```

---

## Problems and fixes

| Problem | Cause | Fix |
|---|---|---|
| `npx convex dev` shows an error about `CLERK_JWT_ISSUER_DOMAIN` | The variable is not set on Convex | Do Step 4, item 6 |
| Sign-in works, but the `users` table is empty | The webhook does not reach Convex | Check the endpoint URL ends with `.convex.site/clerk-webhook`. Check `CLERK_WEBHOOK_SECRET`. Look at **Message Attempts** in the Clerk webhook page |
| The profile page shows no plan and loads forever | Convex does not accept the Clerk token | Check that the Convex integration is active in Clerk and that `CLERK_JWT_ISSUER_DOMAIN` is correct |
| **Start Call** does nothing | A Vapi value is wrong | Check `NEXT_PUBLIC_VAPI_API_KEY` is the **public** key, and check `NEXT_PUBLIC_VAPI_ASSISTANT_ID`. Restart `npm run dev` after you change `.env.local` |
| The call ends, but no plan appears | The tool call failed | Open **Logs** in the Convex dashboard. Look for `Error generating fitness plan`. Also check the call log in Vapi |
| Convex logs show a Gemini error | The key or the model is wrong | Check `GEMINI_API_KEY`. The model name is in `convex/http.ts` |

## Commands

| Command | What it does |
|---|---|
| `npm run dev` | Starts the website on http://localhost:3000 |
| `npx convex dev` | Starts the Convex dev sync and updates `convex/_generated` |
| `npm run build` | Builds the website for production |
| `npm run start` | Starts the production build |
| `npm run lint` | Checks the code with ESLint |

## Project structure

| Folder | Contents |
|---|---|
| `src/app/` | The pages: home, `generate-program`, `profile`, and the Clerk sign-in pages |
| `src/components/` | Shared UI parts. `ui/` holds the shadcn/ui parts |
| `src/providers/` | The Clerk and Convex providers |
| `src/lib/vapi.ts` | The Vapi client |
| `src/middleware.ts` | Makes `/generate-program` and `/profile` private |
| `convex/schema.ts` | The database tables |
| `convex/http.ts` | The Clerk webhook and the plan generator endpoint |
| `convex/plans.ts`, `convex/users.ts` | Database queries and mutations |

## License

MIT. See [LICENSE](LICENSE).
