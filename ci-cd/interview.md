# CI/CD Interview Questions

1. **What is CI/CD?**
   - **CI (Continuous Integration)** is the practice of automatically building and testing code every time a developer pushes a change. **CD (Continuous Delivery/Deployment)** is automatically deploying that validated code to a staging or production environment.

2. **What is the difference between Continuous Delivery and Continuous Deployment?**
   - Continuous Delivery: every change that passes CI is ready to deploy, but a human presses the deploy button. Continuous Deployment: every passing build is automatically deployed to production — no human approval needed.

3. **What are the typical stages in a CI/CD pipeline?**
   - Trigger (push/PR) → Install dependencies → Lint → Run tests → Build → Deploy to staging → (optionally) Deploy to production.

4. **What is GitHub Actions?**
   - GitHub's built-in CI/CD platform. Workflows are defined in YAML files (`.github/workflows/`) and can be triggered by push, pull request, schedule, or manual dispatch. Each workflow has jobs made up of steps.

5. **What is a GitHub Actions runner?**
   - The machine that executes a job. `ubuntu-latest`, `windows-latest`, and `macos-latest` are GitHub's hosted runners. You can also self-host runners on your own machines.

6. **How do you store sensitive values like API keys in CI/CD?**
   - As **Secrets** stored in GitHub repository or organization settings. They are never shown in logs and are injected as environment variables in the workflow:
   ```yaml
   env:
     API_KEY: ${{ secrets.MY_API_KEY }}
   ```

7. **What is a matrix strategy in GitHub Actions?**
   - Running the same job across multiple configurations in parallel:
   ```yaml
   strategy:
     matrix:
       node: [18, 20, 22]
   ```
   This runs three parallel jobs, each with a different Node version.

8. **What is the `needs` keyword in GitHub Actions?**
   - It declares job dependencies. A job with `needs: [lint, test]` only runs after both lint and test jobs complete successfully.

9. **What is artifact sharing between jobs?**
   - Jobs run on separate machines — they can't share files directly. Use `actions/upload-artifact` in one job and `actions/download-artifact` in another to pass build output (like a `dist/` folder) between jobs.

10. **Why is CI/CD important for a development team?**
    - It catches bugs immediately (not days later), ensures the main branch is always deployable, removes manual deployment steps (which cause human error), and gives the team confidence to ship frequently. "If it hurts, do it more often" — automation makes deployment boring and safe.
