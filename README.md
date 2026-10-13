import streamlit as st
import time

# 페이지 기본 설정
st.set_page_config(
    page_title="[데모] AI 동료교사 토론 웹앱",
    page_icon="🗣️",
    layout="wide"
)

# ---------------------------------------------------------
# 세션 상태(Session State) 초기화
# ---------------------------------------------------------
if "step" not in st.session_state:
    st.session_state.step = 1
if "team" not in st.session_state:
    st.session_state.team = None
if "user_name" not in st.session_state:
    st.session_state.user_name = ""
if "role" not in st.session_state:
    st.session_state.role = ""
if "prep_data" not in st.session_state:
    st.session_state.prep_data = {
        "topic": "중학교 생기부에 독서활동 상황을 다시 반영해야 한다",
        "claim": "",
        "rebuttal": "",
        "counter": ""
    }
if "ready_pro" not in st.session_state:
    st.session_state.ready_pro = False
if "ready_con" not in st.session_state:
    st.session_state.ready_con = False

# ---------------------------------------------------------
# STEP 1: 이름 및 팀 코드 입력 화면
# ---------------------------------------------------------
if st.session_state.step == 1:
    st.title("🗣️ 토론 수업 코파일럿 웹앱 (시연용)")
    st.caption("※ 본 웹앱은 API 키가 필요 없는 체험용 데모버전입니다.")
    st.divider()

    col1, col2 = st.columns([1, 1])
    with col1:
        st.subheader("🔑 팀 로그인")
        name_input = st.text_input("이름을 입력하세요:", value=st.session_state.user_name)
        code_input = st.text_input("팀 코드를 입력하세요 (찬성: 5547 / 반대: 3141):", type="password")

        if st.button("입장하기 ➡️", use_container_width=True):
            if not name_input.strip():
                st.error("이름을 입력해 주세요.")
            elif code_input == "5547":
                st.session_state.user_name = name_input
                st.session_state.team = "찬성"
                st.session_state.step = 2
                st.rerun()
            elif code_input == "3141":
                st.session_state.user_name = name_input
                st.session_state.team = "반대"
                st.session_state.step = 2
                st.rerun()
            else:
                st.error("올바른 팀 코드를 입력하세요! (찬성: 5547 / 반대: 3141)")

    with col2:
        st.info("""
        💡 **테스트 가이드**
        * **찬성팀 코드:** `5547`
        * **반대팀 코드:** `3141`
        * 별도의 외부 API 키나 데이터베이스 연결 없이 동작합니다.
        """)

# ---------------------------------------------------------
# STEP 2: 역할 선택 화면
# ---------------------------------------------------------
elif st.session_state.step == 2:
    st.title(f"🎭 [{st.session_state.team}팀] 토론 역할 선택")
    st.write(f"반갑습니다, **{st.session_state.user_name}** 학생!")
    st.divider()

    role_selected = st.radio(
        "토론에서 맡은 주 역할을 선택하세요:",
        ["입론자 (핵심 주장 및 근거 발표)", "반론자 (상대 논지 분석 및 질의)", "최후변론자 (토론 요약 및 최종 입장 발표)"]
    )

    col1, col2 = st.columns([1, 4])
    with col1:
        if st.button("⬅️ 이전"):
            st.session_state.step = 1
            st.rerun()
    with col2:
        if st.button("다음 (개요 작성) ➡️"):
            st.session_state.role = role_selected
            st.session_state.step = 3
            st.rerun()

# ---------------------------------------------------------
# STEP 3: 토론 입론/반박/재반박 내용 입력 화면
# ---------------------------------------------------------
elif st.session_state.step == 3:
    st.title(f"📝 [{st.session_state.team}팀 - {st.session_state.role}] 토론 준비 개요서")
    st.caption("상대팀과 교사가 함께 공유할 핵심 내용을 작성하세요.")
    st.divider()

    topic_val = st.text_input("토론 주제:", value=st.session_state.prep_data["topic"])
    
    col1, col2 = st.columns(2)
    with col1:
        claim_val = st.text_area(
            "1. 우리 팀의 핵심 주장 및 주요 근거:",
            value=st.session_state.prep_data["claim"],
            placeholder="예: 독서 습관을 키우고 사고력을 함양하기 위해 생기부 반영이 필요합니다.",
            height=120
        )
        rebuttal_val = st.text_area(
            "2. 예상되는 상대측의 반박:",
            value=st.session_state.prep_data["rebuttal"],
            placeholder="예: 보여주기식 독서나 사교육 유발 부작용이 생길 수 있다는 반론이 나올 수 있습니다.",
            height=120
        )
    with col2:
        counter_val = st.text_area(
            "3. 상대 반박에 대한 우리 팀의 재반박 카드:",
            value=st.session_state.prep_data["counter"],
            placeholder="예: 검증 방식을 단순 목록 수집이 아닌 수업 연계 서평 작성으로 전환하면 해결 가능합니다.",
            height=250
        )

    col1, col2 = st.columns([1, 4])
    with col1:
        if st.button("⬅️ 이전"):
            st.session_state.step = 2
            st.rerun()
    with col2:
        if st.button("작성 완료 및 대기실 이동 🚀"):
            st.session_state.prep_data = {
                "topic": topic_val,
                "claim": claim_val,
                "rebuttal": rebuttal_val,
                "counter": counter_val
            }
            if st.session_state.team == "찬성":
                st.session_state.ready_pro = True
            else:
                st.session_state.ready_con = True
            st.session_state.step = 4
            st.rerun()

# ---------------------------------------------------------
# STEP 4: 토론 시작 대기 화면 (시뮬레이션 기능 포함)
# ---------------------------------------------------------
elif st.session_state.step == 4:
    st.title("⏳ 토론 시작 대기실")
    st.write(f"주제: **{st.session_state.prep_data['topic']}**")
    st.divider()

    col1, col2 = st.columns(2)
    with col1:
        status_pro = "준비 완료 ✅" if st.session_state.ready_pro else "입력 중... ⏳"
        st.metric(label="찬성팀 준비 상태", value=status_pro)
    with col2:
        status_con = "준비 완료 ✅" if st.session_state.ready_con else "입력 중... ⏳"
        st.metric(label="반대팀 준비 상태", value=status_con)

    st.warning("🤖 **동료교사 AI 안내:** 양쪽 팀이 모두 준비 완료 상태가 되어야 토론을 시작할 수 있습니다.")

    # 데모용 시뮬레이션 토글 버튼 (실제 서버 없이 테스트 가능하도록)
    st.markdown("---")
    st.subheader("🛠️ 데모 시뮬레이션 컨트롤러")
    col_a, col_b = st.columns(2)
    with col_a:
        if st.button("찬성팀 준비완료 토글"):
            st.session_state.ready_pro = not st.session_state.ready_pro
            st.rerun()
    with col_b:
        if st.button("반대팀 준비완료 토글"):
            st.session_state.ready_con = not st.session_state.ready_con
            st.rerun()

    if st.session_state.ready_pro and st.session_state.ready_con:
        st.success("🎉 양팀 모두 준비 완료! 아래 버튼을 눌러 토론을 시작하세요.")
        if st.button("🔥 본격 토론 시작하기", type="primary", use_container_width=True):
            st.session_state.step = 5
            st.rerun()

# ---------------------------------------------------------
# STEP 5: 토론 진행 화면 (타이머 + 요약 카드 + 규칙 가이드)
# ---------------------------------------------------------
elif st.session_state.step == 5:
    st.title(f"⚖️ 실시간 토론: {st.session_state.prep_data['topic']}")
    st.caption(f"접속자: {st.session_state.user_name} ({st.session_state.team}팀 / {st.session_state.role})")
    st.divider()

    # 상단 요약 카드 영역 (사전 입력 내용 보시면서 발표하도록 제시)
    st.subheader("📋 사전 준비 개요 요약 카드")
    col1, col2 = st.columns(2)
    with col1:
        st.info(f"**[{st.session_state.team}팀 작성 카카오]**\n\n"
                f"• **주장/근거:** {st.session_state.prep_data['claim']}\n\n"
                f"• **예상 반박:** {st.session_state.prep_data['rebuttal']}")
    with col2:
        st.success(f"**[상대 반박 대비 재반박 카드]**\n\n"
                   f"• **재반박 방안:** {st.session_state.prep_data['counter']}\n\n"
                   f"• **핵심 단서:** 상대방의 논리적 허점을 짚는 질문을 준비하세요.")

    st.divider()

    # 토론 단계 및 타이머 컨트롤
    st.subheader("⏱️ 토론 진행 절차 및 타이머")

    steps_info = [
        {"stage": "1단계: 찬성측 입론 (3분)", "time": 180, "guide": "찬성 측 입론자가 주장과 근거를 명확히 발표합니다."},
        {"stage": "2단계: 반대측 입론 (3분)", "time": 180, "guide": "반대 측 입론자가 주장과 근거를 명확히 발표합니다."},
        {"stage": "3단계: 자유 반론 및 교차 질의 (5분)", "time": 300, "guide": "상대방의 입론 내용을 바탕으로 질의응답 및 재반박 카드를 활용합니다."},
        {"stage": "4단계: 최후 변론 (2분)", "time": 120, "guide": "각 측 최후변론자가 오늘 토론의 요점을 최종 정리합니다."}
    ]

    selected_stage_name = st.selectbox(
        "현재 진행할 토론 단계를 선택하세요:",
        [s["stage"] for s in steps_info]
    )

    selected_info = next(item for item in steps_info if item["stage"] == selected_stage_name)

    st.notice = st.warning(f"💡 **AI 진행 가이드:** {selected_info['guide']}")

    col1, col2 = st.columns([1, 2])
    with col1:
        start_timer = st.button("⏱️ 타이머 시작", use_container_width=True)
    
    if start_timer:
        t = selected_info["time"]
        timer_display = st.empty()
        while t > 0:
            mins, secs = divmod(t, 60)
            timer_display.metric("⏱️ 남은 시간", f"{mins:02d}:{secs:02d}")
            time.sleep(1)
            t -= 1
        timer_display.error("🔔 제한 시간이 종료되었습니다! 다음 단계로 이동해주세요.")

    st.divider()
    if st.button("🔄 처음으로 돌아가기"):
        st.session_state.step = 1
        st.session_state.ready_pro = False
        st.session_state.ready_con = False
        st.rerun()
