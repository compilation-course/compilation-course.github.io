---
layout: default
title: Example of an optimization pass
parent: Code Generation 
nav_order: 6
---

# Example of an optimization pass

```cpp
#include "llvm/Analysis/ConstantFolding.h"
#include "llvm/IR/Constants.h"
#include "llvm/IR/Instructions.h"
#include "llvm/IR/Module.h"
#include "llvm/Pass.h"
#include "llvm/Support/raw_ostream.h"

#include <set>

using namespace llvm;

namespace {
struct DTigerOpts : public ModulePass {
  static char ID;

  DTigerOpts() : ModulePass(ID) {}

  // Return the function __not if it is used in the module
  Function *find_not_function(Module &M) {
    for (auto &fun : M.getFunctionList())
      if (fun.getName() == "__not") {
        errs() << "Found function __not\n";
        return &fun;
      }
    return nullptr;
  }

  // Given a use of function __not, this function checks if the use
  // corresponds to a call with a constant integer parameter, such as
  // __not(1). In that case, it directly evaluates the expected result
  // and removes the call.
  bool replace_constant_call(User *use, Module &M, const DataLayout &DL) {
    // Retrieve the CallSite of the use

    CallInst *call = dyn_cast<CallInst>(use);
    // Check if it is a direct call instruction
    if (!call)
      return false;
    errs() << "Found call to function __not\n";

    // Since the typechecker has run, we assume that there is exactly one
    // argument
    // on the call to __not
    assert(call->arg_begin() != nullptr);
    Value *arg = call->arg_begin()->get();

    // If the argument is not a constant integer skip to next call
    ConstantInt *c;
    if (!(c = dyn_cast<ConstantInt>(arg)))
      return false;

    // Evaluate the constant call result
    char result = (c->equalsInt(0) ? 1 : 0);

    // Record all uses of the call result
    std::set<Instruction *> WorkList;
    for (auto use : call->users()) {
      if (auto i = dyn_cast<Instruction>(use)) {
        WorkList.insert(i);
      }
    }

    // Replace the call by the constant result
    call->replaceAllUsesWith(
        ConstantInt::getSigned(Type::getInt32Ty(M.getContext()), result));
    call->eraseFromParent();

    // Constant propagate to all users of the call
    while (!WorkList.empty()) {
      Instruction *i = *WorkList.begin();
      WorkList.erase(WorkList.begin());
      if (Constant *c = ConstantFoldInstruction(i, DL)) {
        for (auto use : i->users())
          WorkList.insert(cast<Instruction>(use));
        i->replaceAllUsesWith(c);
        i->eraseFromParent();
      }
    }

    return true;
  }

  bool runOnModule(Module &M) {
    const DataLayout &DL = M.getDataLayout();

    bool modified = false, did_replace;

    // Check if the function __not is used in the module
    Function *not_fun = find_not_function(M);
    if (not_fun == nullptr)
      return false;

    // If __not is referenced, check every use for constant calls
    // that we could optimize
    do {
      did_replace = false;
      for (const auto user : not_fun->users()) {
        if (replace_constant_call(user, M, DL)) {
          did_replace = true;
          errs() << "Found and replaced constant call\n";
        }
      }
      modified |= did_replace;
      // If we did replace some calls to __not; do a further pass
      // to enable successive folds such as not(not(0)) -> 0
    } while (did_replace);

    return modified;
  }
};
} // namespace

char DTigerOpts::ID = 0;
static RegisterPass<DTigerOpts>
    X("dtigeropts", "Optimization pass for dragon tiger", false, false);
```
