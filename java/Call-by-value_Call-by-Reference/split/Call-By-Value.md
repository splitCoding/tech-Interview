# Call-By-Value 

## Java 의 파라미터 전달 방법

자바는 메서드를 호출할 때 파라미터를 전달하는 방법으로 Call By Value 를 사용합니다. <br>
`Call by Value` 는 파라미터를 전달할 때 변수를 복사하여 전달하기 때문에 전달한 변수와 전달받은 변수는 다른 변수입니다.

## 테스트 코드
```java
class CallByValueTest {

    /**
     * 자바는 메서드를 호출할 때 파라미터를 전달하는 방법으로 Call By Value 를 사용한다.
     *
     * 호출된 함수의 인자는 호출한 함수에서 전달한 변수가 복사되어 있는 변수로 서로 다른 변수입니다.
     * 호출된 함수의 파라미터를 수정해도 호출한 함수가 전달한 변수는 변경되지 않습니다.
     */

    private Member ako = new Member("ako");

    @Test
    void test() {
        //원시 타입
        int number = 0;
        //참조 타입
        Member member = new Member("split");

        //number 를 10으로 변경
        changeNumberTo10(number);
        assertThat(number).isNotEqualTo(10);

        //split 멤버 참조변수를 ako 멤버 참조변수로 변경
        changeMemberToAko(member);
        assertThat(member.name).isNotEqualTo(ako.name);

        //매세드내에서 split 멤버 참조변수내의 이름 변경
        changeNickname(member, ako.name);
        assertThat(member.name).isEqualTo(ako.name);
    }

    private void changeNumberTo10(int number) {
        number = 10;
    }

    private void changeMemberToAko(Member member) {
        member = ako;
    }

    private void changeNickname(final Member member, final String newNickname) {
        member.name = newNickname;
    }

    private static class Member {

        protected String name;

        public Member(final String name) {
            this.name = name;
        }
    }
}
```

### JVM 메모리 변수가 저장 위치
Java 에서는 함수에서 사용되는 지역변수, 인자들은 Stack 영역에 저장됩니다.
- 참조 변수는 HEAP 영역에 저장된 뒤에 해당 주소값을 가르키는 변수가 Stack 영역에 저장됩니다.

### 메서드 호출 시 메모리 상태
<div style="text-align: center">
    <img src="./image/call-by-value-before.png" style="text-align: center" width="600">
</div>

### 메서드 호출 후 메모리 상태
<div style="text-align: center">
    <img src="./image/call-by-value-after.png" style="text-align: center" width="600">
</div>

- Call-by-Value의 장점은 값을 복사하기 때문에 원래 값의 불변성을 보장합니다.
- Call-by-Value의 단점은 값을 복사하기 때문에 메모리 사용량이 늘어납니다.
