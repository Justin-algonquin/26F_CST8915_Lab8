# Lab 8 student guide for direct OpenAI access

Use this guide for Step 3 if your Azure for Students subscription cannot access the required Azure AI resources or models. You will still deploy the pet store to AKS. The instructor provides credentials through Brightspace for product descriptions and images using the direct OpenAI API at `api.openai.com`.

## Step 3. Configure the Brightspace credentials

1. Download `secrets.yaml` from Brightspace. Keep it outside every Git repository, for example in your Downloads folder. Use the file as supplied. The two Secret names and key fields are already set.
2. From the lab's `Deployment Files` directory, set the path to your downloaded file in your terminal. Adjust the path if you saved it elsewhere.

   ```bash
   LAB8_SECRET_FILE="$HOME/Downloads/secrets.yaml"
   ```

3. Use the provided AI deployment settings in `aps-all-in-one.yaml` for this path. `USE_AZURE_OPENAI` is `"False"`, and the manifest selects `gpt-4o-mini` for descriptions and `gpt-image-2.6-flare` for images. The two Secret names retain their existing names for compatibility with Brightspace.
4. Continue at [Step 4 in the lab instructions](./Lab8_Instructions.md#step-4-deploy-the-configmap-and-secrets) in the same terminal. That step applies the ConfigMap and both Secrets, and verifies their names without displaying their contents.

## Troubleshooting

| Symptom | What to check |
| --- | --- |
| The AI pod shows `CreateContainerConfigError` | Run `kubectl describe pod <ai-pod-name>`. Check that both named Secrets were applied in the same namespace as the pod. |
| The app reports an invalid credential (401) | Apply the current Brightspace file. If the pod was already running, use `kubectl rollout restart deployment/ai-service` to reload its environment. |
| The app reports unavailable model access (403 or 404) | Check that the app is using the direct OpenAI settings, then contact your instructor with the error and model name. |
| The app reports a rate limit or exhausted quota (429) | Wait briefly before retrying. Contact your instructor if the error persists. |

The credentials are shared with your class. Use them to test the lab, and check Brightspace for the current deadline and credential availability. Keep the file out of repositories, recordings, and screenshots, and do not share it outside the class.

> **Grading policy.** Publishing the populated `secrets.yaml` file in a public repository results in a grade of zero.
