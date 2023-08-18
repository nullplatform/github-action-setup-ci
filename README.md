# Nullplatform Setup CI Composite GitHub Action

You can use the GitHub Action to setup CI process and launch builds required by Nullplatform.

## Change action.yml

The action.yml defines the inputs and output for your action.

Update the action.yml with your name, description, inputs and outputs for your action.

See the [documentation](https://help.github.com/en/articles/metadata-syntax-for-github-actions)

## Package for distribution

Update version in ``update-version.sh`` file changing ``VERSION_NUMBER`` and then run:

```bash
chmod 400 ./update-version.sh
```

```bash
./update-version.sh
```

## Usage

You can now consume the action by referencing the v1 branch

### Using 

```yaml
- name: Start Nullplatform CI
  id: setup-ci
  uses: nullplatform/github-action-setup-ci@v1
  with:
  api-key: ${{ secrets.NULLPLATFORM_API_KEY }}

##
## Other steps
##

- name: End Nullplatform CI
  if: ${{ always() }}
  id: end-setup-ci
  uses: nullplatform/github-action-setup-ci@main
  with:
    build-id: ${{ steps.setup-ci.outputs.build-id }}
    status: ${{ (steps.push-asset.conclusion == 'failure' || steps.push-asset.conclusion == 'cancelled') && 'failed' || 'successful' }}
```
