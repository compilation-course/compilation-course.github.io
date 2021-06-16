---
layout: default 
title: Visitor (ast_dumper.cc) Example 
parent: Lexical and syntax analysis 
nav_order: 3
---

# Visitor (ast_dumper.cc) Example

```cpp
/* src/driver/driver.cc */

#include "../ast/ast_dumper.hh"
#include "../ast/nodes.hh"
#include "../utils/errors.hh"
#include "../utils/nolocation.hh"
#include <iostream>

using namespace ast;

int main(int argc, char **argv) {
  /* Declare an AST Tree */
  Expr *cond_ = new IntegerLiteral(utils::nl, 10);
  Expr *then_ = new BinaryOperator(utils::nl, new IntegerLiteral(utils::nl, 5),
                                   new IntegerLiteral(utils::nl, 7), o_plus);
  Expr *else_ = new IntegerLiteral(utils::nl, 0);
  Node *root = new IfThenElse(utils::nl, cond_, then_, else_);

  /* Print the Tree */
  ASTDumper dumper(&std::cout, true);
  root->accept(dumper);
  dumper.nl();
  return 0;
}

/* src/ast/nodes.hh (partial) */

namespace ast {

typedef enum {
  o_plus = 0,
  o_minus,
  o_times,
  o_divide,
  o_eq,
  o_neq,
  o_lt,
  o_le,
  o_gt,
  o_ge
} Operator;
const std::string operator_name[] = {"+",  "-", "*",  "/", "=",
                                     "<>", "<", "<=", ">", ">="};

class Node {

  // Private fields
  Type type = t_undef;

public:
  // Public fields
  const location loc;

  // Constructor
  Node(const location &_loc) : loc(_loc) {}

  // Destructor
  virtual ~Node() {}

  // Delete copy operator and constructor
  Node &operator=(const Node &) = delete;
  Node(const Node &) = delete;

  // Setter and getters for field `type'
  void set_type(Type _type) {
    assert(type == t_undef && _type != t_undef);
    type = _type;
  }
  Type &get_type() { return type; }
  const Type &get_type() const { return type; }

  // Acceptor method for visitors
  virtual void accept(ASTVisitor &visitor) = 0;
  virtual void accept(ConstASTVisitor &visitor) const = 0;
  virtual llvm::Value *accept(ConstASTValueVisitor &visitor) const = 0;
  virtual int32_t accept(ConstASTIntVisitor &visitor) const = 0;
};

class Expr : public Node {
public:
  // Constructor
  Expr(const location &_loc) : Node(_loc) {}
};

class Decl : public Node {
public:
  // Public fields
  const Symbol name;
  const optional<Symbol> type_name;
  int depth = -1;

  // Constructor
  Decl(const location &_loc, const Symbol &_name,
       const optional<Symbol> &_type_name)
      : Node(_loc), name(_name), type_name(_type_name) {}

  // Setter and getters for field `depth'
  void set_depth(int _depth) {
    assert(depth == -1 && _depth != -1);
    depth = _depth;
  }
  int &get_depth() { return depth; }
  const int &get_depth() const { return depth; }
};

class IntegerLiteral : public Expr {
public:
  // Public fields
  const int32_t value;

  // Constructor
  IntegerLiteral(const location &_loc, const int32_t &_value)
      : Expr(_loc), value(_value) {}

  // Acceptor method for visitors
  virtual void accept(ASTVisitor &visitor) { visitor.visit(*this); }
  virtual void accept(ConstASTVisitor &visitor) const { visitor.visit(*this); }
  virtual llvm::Value *accept(ConstASTValueVisitor &visitor) const {
    return visitor.visit(*this);
  }
  virtual int32_t accept(ConstASTIntVisitor &visitor) const {
    return visitor.visit(*this);
  }
};

class BinaryOperator : public Expr {

  // Private fields
  Expr *left;
  Expr *right;

public:
  // Public fields
  const Operator op;

  // Constructor
  BinaryOperator(const location &_loc, Expr *_left, Expr *_right,
                 const Operator &_op)
      : Expr(_loc), left(_left), right(_right), op(_op) {}

  // Destructor
  virtual ~BinaryOperator() {
    delete right;
    delete left;
  }

  // Getters for field `left'
  Expr &get_left() { return *left; }
  const Expr &get_left() const { return *left; }

  // Getters for field `right'
  Expr &get_right() { return *right; }
  const Expr &get_right() const { return *right; }

  // Acceptor method for visitors
  virtual void accept(ASTVisitor &visitor) { visitor.visit(*this); }
  virtual void accept(ConstASTVisitor &visitor) const { visitor.visit(*this); }
  virtual llvm::Value *accept(ConstASTValueVisitor &visitor) const {
    return visitor.visit(*this);
  }
  virtual int32_t accept(ConstASTIntVisitor &visitor) const {
    return visitor.visit(*this);
  }
};

class IfThenElse : public Expr {

  // Private fields
  Expr *condition;
  Expr *then_part;
  Expr *else_part;

public:
  // Constructor
  IfThenElse(const location &_loc, Expr *_condition, Expr *_then_part,
             Expr *_else_part)
      : Expr(_loc), condition(_condition), then_part(_then_part),
        else_part(_else_part) {}

  // Destructor
  virtual ~IfThenElse() {
    delete else_part;
    delete then_part;
    delete condition;
  }

  // Getters for field `condition'
  Expr &get_condition() { return *condition; }
  const Expr &get_condition() const { return *condition; }

  // Getters for field `then_part'
  Expr &get_then_part() { return *then_part; }
  const Expr &get_then_part() const { return *then_part; }

  // Getters for field `else_part'
  Expr &get_else_part() { return *else_part; }
  const Expr &get_else_part() const { return *else_part; }

  // Acceptor method for visitors
  virtual void accept(ASTVisitor &visitor) { visitor.visit(*this); }
  virtual void accept(ConstASTVisitor &visitor) const { visitor.visit(*this); }
  virtual llvm::Value *accept(ConstASTValueVisitor &visitor) const {
    return visitor.visit(*this);
  }
  virtual int32_t accept(ConstASTIntVisitor &visitor) const {
    return visitor.visit(*this);
  }
};
} // namespace ast

/* src/ast/ast_dumper.cc */

#include "ast_dumper.hh"
#include "../utils/errors.hh"


namespace ast {

void ASTDumper::visit(const IntegerLiteral &literal) {
  *ostream << literal.value;
}

void ASTDumper::visit(const BinaryOperator &binop) {
  *ostream << '(';
  binop.get_left().accept(*this);
  *ostream << operator_name[binop.op];
  binop.get_right().accept(*this);
  *ostream << ')';
}

void ASTDumper::visit(const IfThenElse &ite) {
  *ostream << "if ";
  inl();
  ite.get_condition().accept(*this);
  dnl();
  *ostream << " then ";
  inl();
  ite.get_then_part().accept(*this);
  dnl();
  *ostream << " else ";
  inl();
  ite.get_else_part().accept(*this);
  dec();
}

} // namespace ast
```
