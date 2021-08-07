# SOPS
Github을 통해 배포 파일을 관리하다도면 항상 고민하는 부분은 Secret에 대한 관리
메신저와 같은 방법으로 팀원들과 공유할 수는 있지만, 같은 secret key를 사용하도록 관리하는 것/변경 사항에 대해서도 관리하기 어렵다는 불편함이 존재

## [SOPS?](https://github.com/mozilla/sops)
암호화된 파일을 위한 editor로 YAML, JSON, INI, Binary file의 format을 지원

### 사용 방법
AWS/GCP KMS, Valut 등 KMS 들을 지원, 직접 PGP Key를 만들어서 사용 가능

#### Import
```bash
$ gpg --import php-key.asc
```

### Edit
```bash
$ sops <filename>
```
#### Encrypt
암호화 할 때에는 PGP Fingerprint가 필요 or pgp key 설정을 `.sops.yaml`에 추가해야한다.
```bash
$ sops --pgp <pgp fingerprint> --encrypt <filename>
```

#### Decrypt
```bash
$ sops --decrypt <filename>
```

#### Tip!
- source 파일의 확장자에 맞게 암호화, 복호화를 진행한다. (e.g. `yaml` => `yaml` / `json` => `json`)
- git을 사용할 떄, sops를 통해 변경된 파일의 diff를 확인하기 위해서 `.gitattributes`와 `.gitconfig`를 변경하면 원본 diff도 확인이 가능하다.
