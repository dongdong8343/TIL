패스워드를 검증하는 로직이 기존에는 Validator라는 클래스를 만들어서 해당 클래스 안에 존재했었다.

Validator 클래스에는 이메일, 비밀번호 등 다양한 유저와 관련된 검증 로직이 담겨져 있었다.

하지만 이번에 리팩토링을 진행하면서 Password와 관련된 로직들은 하나의 클래스로 모아서 유지보수성, 응집도를 높이고 싶었다.

그래서 Password 클래스를 별도로 만들어 비밀번호와 관련된 로직을 해당 클래스 내부로 모두 모았다.

이를 통해 검증 책임이 Password 클래스 안에 존재하고 나중에 유지보수를 할 때 비밀번호와 관련된 사항들은 Password 클래스만 수정하면 된다.

코드는 아래와 같다. 그리고 아래 클래스를 엔티티의 멤버로 가지고 있는 형태로 작성했다.

```java
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Embeddable
@Getter
public class Password {
	private static final int MIN_PASSWORD_LENGTH = 8;
	private static final int MIN_PASSWORD_COMPLEXITY = 2;
	private static final int BAD_SEQUENCE_LENGTH = 4;
	private static final String[] BAD_SEQUENCES = {
		"abcdefghijklmnopqrstuvwxyz", "qwertyuiop", "asdfghjkl", "zxcvbnm", "0123456789"
	};
	private static final String SPECIAL_CHAR_REGEX = ".*[!@#$%^&*()_+\\-={}|\\[\\]:\";'<>?,./`~].*";
	private static final String[] COMPLEXITY_PATTERNS = {
		".*[A-Z].*",           // 대문자
		".*[a-z].*",           // 소문자
		".*[0-9].*",           // 숫자
		SPECIAL_CHAR_REGEX     // 특수문자
	};

	private String password;

	private Password(String password) {
		this.password = password;
	}

	public static Password fromRaw(BCryptPasswordEncoder bCryptPasswordEncoder, String password) {
		validate(password);

		return new Password(bCryptPasswordEncoder.encode(password));
	}

	// 비밀번호 맞는지 확인
	private static void validate(String password) {
		if (StringUtils.isBlank(password) || password.length() < MIN_PASSWORD_LENGTH) {
			throw new PasswordTooShortException();
		}

		long typeCount = Arrays.stream(COMPLEXITY_PATTERNS)
			.filter(password::matches)
			.count();

		if (typeCount < MIN_PASSWORD_COMPLEXITY) {
			throw new PasswordTooSimpleException();
		}

		checkBadSequence(password.toLowerCase());
	}

	private static void checkBadSequence(String lowerPassword) {
		for (String seq : BAD_SEQUENCES) {
			for (int i = 0; i <= seq.length() - BAD_SEQUENCE_LENGTH; i++) {
				String subSeq = seq.substring(i, i + BAD_SEQUENCE_LENGTH);
				if (lowerPassword.contains(subSeq)) {
					throw new PasswordHasSequenceException();
				}
			}
		}
	}

	// 비밀번호 동일한지 확인하는 메서드
	public void matches(BCryptPasswordEncoder encoder, String storedPassword) {
		if (!encoder.matches(password, storedPassword)) {
			throw new InvalidPasswordException();
		}
	}

	// 비밀번호 변경 시 이전 비밀번호와 같은지 비교 (같다면 예외)
	public void checkSameAs(BCryptPasswordEncoder encoder, String storedPassword) {
		if (encoder.matches(password, storedPassword)) {
			throw new SameAsPreviousPasswordException();
		}
	}
}

```
