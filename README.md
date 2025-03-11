# Mixin TypeScript Solution

This package provides a solution for combining classes in TypeScript via a "mixin", while preserving the generic types of the involved classes. The solution is fully **type-safe**, ensuring that the generic types from both the base class and the mixin are properly maintained in the resulting class.

## What does this code do?

It allows you to combine classes that have constructors with different generic types, ensuring that the resulting types are correctly merged in the combined class.

### Key Features:

- **Preservation of generic types**: The generic types of the base class and the mixin are merged without losing precision.
- **Fully type-safe**: The solution ensures that the code works safely from a TypeScript typing perspective.
- **Compatibility with complex classes**: Works even with classes that require complex arguments for their constructors.

```typescript

export function mixin<T1, TRes1, T2, TRes2>(
	base: Constructor<T1, TRes1>,
	mixin: Constructor<T2, TRes2>,
): Mixin<T1, TRes1, T2, TRes2> {
	const Base = base as Constructor<any, any>

	Object.getOwnPropertyNames(mixin.prototype).forEach(name => {
		Object.defineProperty(
			Base.prototype,
			name,
			Object.getOwnPropertyDescriptor(mixin.prototype, name) || Object.create(null),
		)
	})

	return class MixedClass extends Base {
		constructor(arg: [T1], arg2: [T2]) {
			super(...arg)
			const mixinInstance = new mixin(...arg2)
			Object.assign(this, mixinInstance)
		}
	} as Mixin<T1, TRes1, T2, TRes2>
}

class Example1<T extends number> {
	id!: number

	constructor(arg: { id: T }) {
		this.id = arg.id
	}
}

class Example2<T extends Record<string, any>> {
	obj!: T
	constructor(arg: T) {
		this.obj = arg
	}
}

export type Constructor<TArgs = any, TRes = any> = new (...args: TArgs[]) => TRes

export type Mixin<T1, TRes1, T2, TRes2> = new (arg: [T1], arg2: [T2]) => TRes1 & TRes2

const Mixed = mixin(Example1, Example2)
//   ^? new <T extends number, T1 extends Record<string, any>>(arg: [{ id: T;}], arg2: [T1]) => Example1<T> & Example2<T1>

const result = new Mixed([{ id: 123 }], [{ name: 'noop' }])
//   ^? Example1<123> & Example2<{ name: string; }>

console.log(result)
// ^ MixedClass { id: 123, obj: { name: "noop" } }
