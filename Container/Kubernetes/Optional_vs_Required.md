k8s resource spec은 optional or required가 될 수 있다.
- optional (`+optional`)
	- `omitempty` struct tag와 같이 쓰이는게 보통이지만 empty value로 제공하는 것에는 사용되면 안됨
	- Pointer Value이거나 build-in nil value(map & slice)
	
- required
	- pointer type X

pointer type은 unset과 zero value를 구분하기 위해서 사용
- zero value가 금지되어 있거나 unset을 암시하기 때문에 optional에서는 필요 없다. 하지만 구현할 때 empty와 zero value를 구분하기 어렵고, struct가 omitempty가 설정됨에도 불구하고 encoder output에서 생략되지 않기 때문에 zero value와 혼동된다. 암시적으로 pointer는 optional임을 약속하고 사용한다 ( built-in `nil` value가 없는 경우에)

#### Reference
- https://github.com/crossplane/crossplane/blob/1fb67d0526b79cefb9949cbfc9c103cdd228c3fd/design/one-pager-managed-resource-api-design.md#pointer-types-and-markers

- https://github.com/kubernetes/community/blob/master/contributors/devel/sig-architecture/api-conventions.md#optional-vs-required
