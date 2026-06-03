# Training Contracts

Shared protobuf contracts for the training platform microservices.

## Structure

```
proto/
├── common/          # Common types (Empty, IdRequest)
├── auth/            # Auth service contracts
├── training/        # Training service contracts
└── ai/              # AI service contracts
```

## Usage as submodule

```bash
git submodule add https://github.com/Olwynion/training-contracts.git proto/contracts
```

Then update submodule in each service:

```bash
git submodule update --init --recursive
```
