import streamlit as st
import pandas as pd
from tkcalendar import DateEntry

def is_status_column(column_name):
    """Xác định cột có phải là trạng thái không"""
    return "status" in column_name.lower()

def load_file():
    uploaded_file = st.file_uploader("Chọn File Excel", type=["xlsx"])
    if uploaded_file:
        return pd.read_excel(uploaded_file)
    return None

def display_table(df):
    if df is not None:
        st.dataframe(df)

        # Thêm các ô nhập cho thêm dòng
        with st.form(key="add_row_form"):
            new_row = []
            for col in df.columns:
                if is_status_column(col):  # Nếu cột là Status, dùng Combobox
                    new_value = st.selectbox(f"Chọn {col}", ["In Progress", "Done", "Cancel"], key=col)
                else:
                    new_value = st.text_input(f"Nhập {col}", key=col)
                new_row.append(new_value)

            submit_button = st.form_submit_button(label="Thêm dòng mới")
            if submit_button:
                df.loc[len(df)] = new_row
                st.success("Dòng mới đã được thêm!")

        # Chỉnh sửa, xóa dòng
        selected_row = st.selectbox("Chọn dòng để chỉnh sửa", df.index, format_func=lambda x: f"Row {x}")
        selected_values = df.loc[selected_row]

        with st.form(key="edit_row_form"):
            edit_row = []
            for idx, col in enumerate(df.columns):
                if is_status_column(col):  # Nếu cột là Status, dùng Combobox
                    edited_value = st.selectbox(f"Sửa {col}", ["In Progress", "Done", "Cancel"], index=["In Progress", "Done", "Cancel"].index(selected_values[col]), key=f"{col}_edit")
                else:
                    edited_value = st.text_input(f"Sửa {col}", value=selected_values[col], key=f"{col}_edit")
                edit_row.append(edited_value)

            edit_button = st.form_submit_button(label="Lưu chỉnh sửa")
            if edit_button:
                df.loc[selected_row] = edit_row
                st.success(f"Dòng {selected_row} đã được chỉnh sửa!")

        # Xóa dòng
        delete_row = st.button("Xóa dòng đã chọn")
        if delete_row:
            df.drop(selected_row, inplace=True)
            st.success(f"Dòng {selected_row} đã được xóa.")

        # Lưu file
        if st.button("Lưu File"):
            st.write("Lưu file đã chỉnh sửa...")
            df.to_excel("Updated_Buyoff_Tracker.xlsx", index=False)
            st.success("File đã được lưu thành công!")

# Main Streamlit app
def main():
    st.title("Buyoff Test Fixture")

    # Tải và hiển thị file
    df = load_file()
    display_table(df)

if __name__ == "__main__":
    main()
