# Lambda Adaptation Strategy - Node.js Express BYOL

## Strategy Selected: A - serverless-http Adapter

### Why This Strategy?

1. **Minimal code changes**: Only 1 line to wrap the app
2. **No app.js modifications**: Express app remains completely Lambda-unaware
3. **Excellent cold-start performance**: ~200-400ms
4. **Full HTTP/2 support**: Works perfectly with API Gateway v2 format
5. **Battle-tested**: Production-ready, widely used
6. **Better v2 format support**: Handles API Gateway v2 payload format correctly

### Implementation Details

- **Wrapper file**: `lambda.js` (minimal wrapper)
  - Imports `app` from `app.js`
  - Wraps it with `serverless-http` adapter
  - Exports Lambda handler as `module.exports.handler`
  
- **Dependencies**: `serverless-http` (npm package, added to package.json)
  
- **Handler value in template.yaml**: `lambda.handler`

### How It Works

1. API Gateway sends HTTP request → Lambda event (format: 2.0)
2. `serverless-http` adapter:
   - Translates Lambda event → Node.js req object
   - Translates res → Lambda response
   - Handles header/body encoding
3. Express app processes request normally
4. Response sent back through API Gateway

### Testing Strategy

1. **Local test** (Step 0):
   ```bash
   npm install
   npm start
   # Verify: GET /, GET /api/hello/:name, POST /api/echo
   ```

2. **SAM build & deploy** (Step 3):
   ```bash
   sam build
   sam deploy --guided  # first time
   sam deploy           # subsequent
   ```

3. **Live smoke test** (Step 4):
   ```bash
   export API=$(aws cloudformation describe-stacks \
     --stack-name byol-node-express --region us-west-2 \
     --query 'Stacks[0].Outputs[?OutputKey==`ApiUrl`].OutputValue' --output text)
   curl $API
   curl $API/api/hello/Lan
   curl -X POST $API/api/echo -H 'Content-Type: application/json' -d '{"hi":"there"}'
   ```

4. **Cold-start measurement** (Step 5):
   ```bash
   sam logs --stack-name byol-node-express --region us-west-2 -t
   # Find REPORT line → Init Duration = cold start time
   ```

### Deployment Region
**MUST use `us-west-2`** (pre-configured in samconfig.toml)

### Expected Cold-start Result
Will be recorded here after first Lambda invocation.

## File Changes Summary

| File | Change | Notes |
|------|--------|-------|
| `lambda.js` | Already created | No changes needed |
| `app.js` | None | Remains completely Lambda-unaware ✓ |
| `server.js` | None | Local dev still works with `npm start` |
| `package.json` | Already has dependency | No changes needed |
| `template.yaml` | Handler: lambda.handler | Updated |

---
**Status**: Ready for local testing and Lambda deployment
<img width="1918" height="892" alt="image" src="https://github.com/user-attachments/assets/01ea6da4-ff99-4d8a-a8af-dba71ab340d7" />

